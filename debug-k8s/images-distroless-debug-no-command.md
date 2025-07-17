
# 1.Tình huống
Một ngày đẹp trời pod bị lỗi. Bạn thử ngay lệnh "kubectl exec -it ..." vào pod kiểm tra.

Nhưng quãi đạn, pod không hỗ trợ bất kỳ 1 command nào:
- Không có /bin/bash, /bin/sh...???
- Không có ls, cd...???
- Không có netstat, telnet, traceroute...???

Ngớ người đặt dấu chấm hỏi, hóa ra pod này dùng images distroless siêu bảo mật (https://github.com/GoogleContainerTools/distroless). Nó giúp gia tăng độ an toàn cho pod tránh khỏi các cuộc tấn công leo thang khi chiếm được quyền điều khiển Pod.

Vậy admin k8s cluster phải làm sao??? -> bạn vẫn còn 1 cứu tinh. Đó chính là "kubectl debug" để tạo ra "Ephemeral Container" trong pod đang chạy.

# 2.Xây dựng

Bước 1: Tạo pod distroless images

`vim HelloJava.java`

```java
package examples;

public class HelloJava {
    public static void main(String[] args) {
        System.out.println("Hello world");
        int number = 0;

        while (true) {
            System.out.println(number);
            number++;

            try {
                Thread.sleep(1000); // Nghỉ 1 giây (1000 milliseconds)
            } catch (InterruptedException e) {
                e.printStackTrace();
            }
        }        
    }
}
```
Thực hiện build image bằng Dockerfile như sau

`vim Dockerfile`
```bash
FROM openjdk:17-jdk-slim AS build-env
COPY . /app/examples
WORKDIR /app
RUN javac examples/*.java
RUN jar cfe main.jar examples.HelloJava examples/*.class 

FROM gcr.io/distroless/java17-debian12
COPY --from=build-env /app /app
WORKDIR /app
CMD ["main.jar"]
```

`docker build -t java-less:v1 .`
```console
=> [build-env 2/5] COPY . /app/examples                                                                                                                                                                 3.8s
=> [build-env 3/5] WORKDIR /app                                                                                                                                                                         0.1s
=> [build-env 4/5] RUN javac examples/*.java                                                                                                                                                            3.1s
=> [build-env 5/5] RUN jar cfe main.jar examples.HelloJava examples/*.class                                                                                                                             0.6s
=> [stage-1 2/3] COPY --from=build-env /app /app                                                                                                                                                        0.1s
=> [stage-1 3/3] WORKDIR /app                                                                                                                                                                           0.1s
=> exporting to image                                                                                                                                                                                   0.1s
=> => exporting layers                                                                                                                                                                                  0.1s
=> => writing image sha256:417e708c31976902                                                                                                             0.0s
=> => naming to docker.io/library/java-less:v1   
```


Sau khi build được images, ta thực hiện đẩy lên docker-hub, private-registry hoặc đẩy bằng docker save , import ctr. Với 2 cách đầu quá thường xuyên làm rồi. Tôi sẽ test cách 3.
```bash
# Save image ra file và scp sang các server worker-node
docker image save docker.io/library/java-less:v1 > java-less.tar
scp java-less.tar ${IP_OF_WORKER_NODE}
# Import local images vào k8s
ctr -n=k8s.io images ls
ctr -n=k8s.io images import java-less.tar
ctr -n=k8s.io images ls | grep java
#(Kết quả):  docker.io/library/java-less:v1
```
Ta thực hiện tạo deployment.yaml với imagePullPolicy: Never để load image trực tiếp từ worker-node (không tải từ docker-hub hoặc private registry).
```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  labels:
    app: app1
  name: app1
spec:
  replicas: 2
  selector:
    matchLabels:
      app: app1
  template:
    metadata:
      labels:
        app: app1
    spec:
      containers:
      - image: docker.io/library/java-less:v1
        name: java-less
        imagePullPolicy: Never
```

```bash
kubectl apply -f deployment.yaml
kubectl get pod
#NAME                    READY   STATUS    RESTARTS   AGE
#app1-686584fffd-gsmw8   1/1     Running   0          3s
#app1-686584fffd-lm88h   1/1     Running   0          3s
```

# 3.Test tình huống


```bash
# Không tồn tại /bin/bash
kubectl exec -it app1-686584fffd-gsmw8 -- /bin/bash
#error: Internal error occurred: Internal error occurred: error executing command in container: failed to exec in container: failed to start exec "": OCI runtime exec failed: exec failed: unable to start container process: exec: "/bin/bash": stat /bin/bash: no such file or directory: unknown

# Không tồn tại /bin/sh
kubectl exec -it app1-686584fffd-gsmw8 -- /bin/sh
#OCI runtime exec failed: exec failed: unable to start container process: exec: "/bin/sh": stat /bin/sh: no such file or directory: unknown

# Chỉ hỗ trợ đúng lệnh java
kubectl exec -it app1-686584fffd-gsmw8 -- "java" "-version"
#openjdk version "17.0.15" 2025-04-15

```
### Vậy làm sao để test netstat, ping, ps???
`kubectl debug -it $TÊN_POD --target=$TÊN_CONAINER --image=$TÊN_IMAGES_BUSYBOX`

```bash
kubectl debug -it app1-686584fffd-gsmw8 --target=java-less --image=busybox

# ps fauxww
PID   USER     TIME  COMMAND
    1 root      0:00 /usr/bin/java -jar main.jar
  118 root      0:00 sh
  124 root      0:00 ps fauxww


# lsof -p 1
1       /usr/lib/jvm/java-17-openjdk-amd64/bin/java     0       /dev/null
1       /usr/lib/jvm/java-17-openjdk-amd64/bin/java     1       pipe:[580092]
1       /usr/lib/jvm/java-17-openjdk-amd64/bin/java     2       pipe:[580093]
1       /usr/lib/jvm/java-17-openjdk-amd64/bin/java     3       /usr/lib/jvm/java-17-openjdk-amd64/lib/modules
1       /usr/lib/jvm/java-17-openjdk-amd64/bin/java     4       /app/main.jar
118     /bin/sh 0       /dev/pts/0
118     /bin/sh 1       /dev/pts/0
118     /bin/sh 2       /dev/pts/0
118     /bin/sh 10      /dev/tty


# ping 8.8.8.8
PING 8.8.8.8 (8.8.8.8): 56 data bytes
64 bytes from 8.8.8.8: seq=0 ttl=111 time=27.020 ms
64 bytes from 8.8.8.8: seq=1 ttl=111 time=27.246 ms
```