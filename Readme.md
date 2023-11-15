# Kubernetes (K8s)
Kubernetes (còn gọi là k8s) là một hệ thống để chạy, quản lý, điều phối các ứng dụng được container hóa trên một cụm máy (1 hay nhiều) gọi là cluster. Với Kubernetes bạn có thể cấu hình để chạy các ứng dụng, dịch vụ sao cho phù hợp nhất khi chúng tương tác với nhau cũng như với bên ngoài. Bạn có thể điều chỉnh tăng giảm tài nguyên, bản chạy phục vụ cho dịch vụ (scale), bạn có thể cập nhật (update), thu hồi update khi có vấn đề ... Kubernetes là một công cụ mạnh mẽ, mềm dẻo, dễ mở rộng khi so sánh nó với công cụ tương tự là Docker Swarm!
#### Khải niệm cơ bản:
- Master Server: là máy chính của cluster, tại đây điều khiển cả cụm máy.
- etct: là thành phần cơ bản cần thiết cho Kubernetes, nó lưu trữ các cấu hình chung cho cả cụm máy, etct chạy tại máy master. etct là một dự án nguồn mở nó cung cấp dịch vụ lưu dữ liệu theo cặp key/value
- kube-apiserver: chạy tại máy master, cung cấp các API Restful để các client (như kubectl) tương tác với Kubernetes
- kube-scheduler: chạy tại master, thành phần này giúp lựa chọn Node nào để chạy các ứng dụng căn cứ vào tài nguyên và các thành phần khác sao cho hệ thống ổn định.
- kube-controller: chạy tại master, nó điều khiển trạng thái cluster, tương tác để thực hiện các tác vụ tạo, xóa, cập nhật ... các tài nguyên
- Kubelet: dịch vụ vụ chạy trên tất cả các máy (Node), nó đảm đương giám sát chạy, dừng, duy trì các ứng dụng chạy trên node của nó.
- Kube-proxy: cung cấp mạng proxy để các ứng dụng nhận được traffic từ ngoài mạng vào cluster.

![](/images//1.png)

## I. Setup Kubernestes on local with VirtualBox and Vagrant

### Điều kiện
- Cài đặt Virtual Box: https://www.virtualbox.org/wiki/Downloads
- Cài đặt Vagrant để config tự động tạo máy chủ trên Virtual Box: https://developer.hashicorp.com/vagrant/install
- Biết dùng Docker, lệnh Linux và Kuberctl cơ bản.
- Cài đặt Docker Desktop và enable Kubernetes trên Docker Desktop.
- Kiểm tra Kubernetes có trên máy bằng lệnh
```
# Lấy thông tin Cluster
kubectl cluster-info

# Các Node có trong Cluster
kubectl get nodes
```
### Tạo Cluster Kubernetes với 2 nodes
- Tạo một Cluster Kubernetes với 3 máy (3 VPS hoặc 3 Server) chạy Centos7.
#### Thông tin các máy của hệ thống
```
1. Máy master: master.nth
- Tên máy: master.nth
- Thông tin hệ thống: HDH Centos7, Docker CE, Kubenestes
- IP: 172.16.10.100
- id: root - pass: 123
- Vai trò: master 

2. Máy worker: worker1.nth
- Tên máy: worker1.nth
- Thông tin hệ thống: HDH Centos7, Docker CE, Kubenestes
- IP: 172.16.10.101
- id: root - pass: 123
- Vai trò: worker 

3. Máy worker: worker2.nth
- Tên máy: worker2.nth
- Thông tin hệ thống: HDH Centos7, Docker CE, Kubenestes
- IP: 172.16.10.102
- id: root - pass: 123
- Vai trò: worker 
```
- Sử dụng Vagrant để tạo hệ thống với 3 máy ảo trên Virtual Box.

#### Step 1. Tạo máy master kubernetes
- cd vào thư mục /kubernetes-centos7/master
- Trong thư mục master có file vagrantfile chưa cấu hình cho máy ảo master.nth.
- Mở virtual Box và chạy lệnh bên dưới tự động tạo máy ảo maste.nth
```
vagrant up
```
(*) Lưu ý: trong quá trình chạy `vagrant up` nếu gặp lỗi `ermission denied (publickey,gssapi-keyex,gssapi-with-mic)` thì chạy lệnh:
```
vagrant ssh -- -vvv
```
để fix lỗi sau đó chạy lại lệnh `vagrant up`

(*) Lưu ý: nhớ kiểm tra máy có thể chạy được lệnh bên dưới không
```
chmode +x install-docker-kube.sh
```
#### Step 2. Tạo các máy woker kubernetes
- cd vào thư mục /kubernetes-centos7/woker1 và /kubernetes-centos7/woker2 để tạo 2 máy worker
- Trong thư mục woker1 và worker2 có file vagrantfile chưa cấu hình cho máy ảo worker.
- thực hiện lệnh:
```
vagrant up
```
để khởi tạo 2 máy worker.

#### Step 3. Khởi tạo Cluster trên máy master.nth
- Trong lệnh khởi tạo cluster có tham số --pod-network-cidr để chọn cấu hình mạng của POD, do dự định dùng Addon calico nên chọn --pod-network-cidr=192.168.0.0/16
- Gõ lệnh sau để khở tạo là nút master của Cluster
```
kubeadm init --apiserver-advertise-address=172.16.10.100 --pod-network-cidr=192.168.0.0/16
```
(*) Lưu ý: trường hợp khởi tạo nút master của cluster gặp lỗi như sau:
```
[preflight] Running pre-flight checks error execution phase preflight: [preflight] Some fatal errors occurred: [ERROR CRI]: container runtime is not running: output: time="2020-09-24T11:49:16Z" level=fatal msg="getting status of runtime failed: rpc error: code = Unimplemented desc = unknown service runtime.v1alpha2.RuntimeService
```
lỗi này xảy ra là vì version mới của kubernetes. version kubectl hơn 1.27.0 nên để fix lỗi này thì ssh vào máy master.nth và chạy lệnh 
```
sudo yum downgrade kubeadm-1.16.9 kubernetes-cni-0.7.5 kubelet-1.16.9 kubectl-1.16.9
```
để down version kubernates xuống sẽ fix được lỗi

- Sau khi lệnh chạy xong, chạy tiếp cụm lệnh nó yêu cầu chạy sau khi khởi tạo- để chép file cấu hình đảm bảo trình kubectl trên máy này kết nối Cluster
```
mkdir -p $HOME/.kube
sudo cp -i /etc/kubernetes/admin.conf $HOME/.kube/config
sudo chown $(id -u):$(id -g) $HOME/.kube/config
```
- Tiếp đó, nó yêu cầu cài đặt một Plugin mạng trong các Plugin tại addon, ở đây đã chọn calico, nên chạy lệnh sau để cài nó
```
kubectl apply -f https://docs.projectcalico.org/archive/v3.12/manifests/calico.yaml

```
- Gõ vài lệnh sau để kiểm tra
```
# Thông tin cluster
kubectl cluster-info
# Các node trong cluster
kubectl get nodes
# Các pod đang chạy trong tất cả các namespace
kubectl get pods -A
```

-> Vậy là đã có Cluster với 1 node!

#### Cấu hình kubectl máy trạm truy cập đến các Cluster
- Chương trình client kubectl là công cụ dòng lệnh kết nối và tương tác với các Cluster Kubernetes, thường khi cài đặt Kubernetes mọi người cũng cài luôn kubectl như phần trên trên, ngay cả máy cài Docker Desktop cũng đã có kubectl. Tất nhiên, bạn có cài đặt kubectl trên một máy không Docker, không Kubernetes với mục đích chỉ dùng nó kết nối đến hệ thống Cluster từ xa. Nếu muốn cài ở máy độc lập như vậy xem tại: https://kubernetes.io/docs/tasks/tools/

- Trở lại máy Host (máy local), để xem nội dung cấu hình kubectl gõ lệnh:
```
kubectl config view
```
- Tại máy master ở trên, có file cấu hình cho tại /root/.kube/config, ta copy file cấu hình này ra lưu thành file config-mycluster (không ghi đè vào config hiện tại của máy HOST)
```
scp root@172.16.10.100:/etc/kubernetes/admin.conf ~/.kube/config-mycluster
```
- Vậy trên máy của tôi đang có 2 file cấu hình:
```
~/.kube/config-mycluster cấu hình kết nối đến Cluster mới tạo ở trên

~/.kube/config cấu hình kết nối đến Cluster cục bộ của bản Kubernetes có sẵn của Docker
```
- Giờ bạn sẽ thực hiện kết hợp 2 file: config và config-mycluster thành 1 và lưu trở lại config.
```
export KUBECONFIG=~/.kube/config:~/.kube/config-mycluster
kubectl config view --flatten > ~/.kube/config_temp
mv ~/.kube/config_temp ~/.kube/config
```
- Như vậy trong file cấu hình đã có các ngữ cảnh khác nhau để sử dụng, đóng terminal và mở lại rồi gõ lệnh, có các ngữ cảnh nào
-  nếu muốn chuyển làm việc sang context có tên kubernetes-admin@kubernetes (nối với cluster mới tạo ở trên) thì gõ lệnh
```
kubectl config use-context kubernetes-admin@kubernetes
```

#### Các node join vào cluster
- Hãy vào máy node master (bằng SSH ssh root@172.16.10.100). Thực hiện lệnh sau với Cluster để lấy lệnh kết nối
```
kubeadm token create --print-join-command
```
- Nó cho nội dung lệnh kubeadm join ... thực hiện lệnh này trên các node worker thì node worker sẽ nối vào Cluster
- SSH vào máy worker1, work2 và thực hiện kết nối: bằng cách pass đoạn token lấy từ lệnh `kubeadm token create --print-join-command` ở máy master

=> Giờ kiểm tra các node có trong Cluster
```
kubectl get nodes
```



### Tổng kết lại
- Đến đây bạn đã biết khởi tạo một Cluster từ Docker Destop hay một Cluster phức tạp 3 node thực thụ, tuy nhiên quá trình cài đặt vẫn chưa hoàn thành, các công cụ cần để dễ dàng làm việc với Kubernetes sẽ tiếp tục ở bài sau, nhưng hiện giờ bạn đã biết các lệnh:
```
# khởi tạo một Cluster
kubeadm init --apiserver-advertise-address=172.16.10.100 --pod-network-cidr=192.168.0.0/16

# Cài đặt giao diện mạng calico sử dụng bởi các Pod
kubectl apply -f https://docs.projectcalico.org/v3.10/manifests/calico.yaml

# Thông tin cluster
kubectl cluster-info

# Các node (máy) trong cluster
kubectl get nodes
kubectl get node -o wide

# Các pod (chứa container) đang chạy trong tất cả các namespace
kubectl get pods -A

# Xem nội dung cấu hình hiện tại của kubectl
kubectl config view

# Thiết lập file cấu hình kubectl sử dụng cho 1 phiên làm việc hiện tại của termianl
export KUBECONFIG=/Users/xuanthulab/.kube/config-mycluster

# Gộp file cấu hình kubectl
export KUBECONFIG=~/.kube/config:~/.kube/config-mycluster
kubectl config view --flatten > ~/.kube/config_temp
mv ~/.kube/config_temp ~/.kube/config

# Các ngữ cảnh hiện có trong config
kubectl config get-contexts

# Đổi ngữ cảnh làm việc (kết nối đến cluster nào)
kubectl config use-context kubernetes-admin@kubernetes

# Lấy mã kết nối vào Cluster
kubeadm token create --print-join-command

# node worker kết nối vào Cluster
kubeadm join 172.16.10.100:6443 --token 5ajhhs.atikwelbpr0 ...

kubectl get rs -o wide
kubectl get hpa -o wide
kubectl get po -o wide
kubectl get nodes -o -wide

kubectl get all -o wide

kubectl apply -f <file>

kubectl delete -f <file>

k9s
```
- https://jamesdefabia.github.io/docs/user-guide/kubectl/kubectl/

## Error
```
@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@
@    WARNING: REMOTE HOST IDENTIFICATION HAS CHANGED!     @
@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@
```

fix

```
ssh-keygen -R 192.168.3.10
```

```
rm /etc/containerd/config.toml
systemctl restart containerd
systemctl enable containerd 
kubeadm init
```

## Kubernetes Dashboard
- https://github.com/kubernetes/dashboard/releases/tag/v2.0.0-beta6
> curl https://github.com/kubernetes/dashboard/releases/tag/v2.0.0-beta6/aio/deploy/recommended.yaml > dashboard-v2.yaml

```
 kubectl apply -f .\dashboard-v2-beta6.yaml

 kubectl get po -n kubernetes-dashboard
```

## ReplicaSet
- ReplicaSet là một điều khiển Controller - nó đảm bảo ổn định các nhân bản (số lượng và tình trạng của POD, replica) khi đang chạy.
- Khi định nghĩa một ReplicaSet (định nghĩa trong file .yaml) gồm các trường thông tin, gồm có trường selector để chọn ra các các Pod theo label, từ đó nó biết được các Pod nó cần quản lý(số lượng POD có đủ, tình trạng các POD). Trong nó nó cũng định nghĩa dữ liệu về Pod trong spec template, để nếu cần tạo Pod mới nó sẽ tạo từ template đó. Khi ReplicaSet tạo, chạy, cập nhật nó sẽ thực hiện tạo / xóa POD với số lượng cần thiết trong khai báo (repilcas).
- Ví dụ trong folder exams/2.replicaset

## HPA - Horizontal Pod Autoscaler
- Horizontal Pod Autoscaler là chế độ tự động scale (nhân bản POD) dựa vào mức độ hoạt động của CPU đối với POD, nếu một POD quá tải - nó có thể nhân bản thêm POD khác và ngược lại - số nhân bản dao động trong khoảng min, max cấu hình

- Ví dụ, với ReplicaSet rsapp trên đang thực hiện nhân bản có định 3 POD (replicas), nếu muốn có thể tạo ra một HPA để tự động scale (tăng giảm POD) theo mức độ đang làm việc CPU, có thể dùng lệnh sau:
```
kubectl autoscale rs rsapp --max=2 --min=1
```
- Lệnh trên tạo ra một hpa có tên rsapp, có dùng tham chiếu đến ReplicaSet có tên rsapp để scale các POD với thiết lập min, max các POD
- Để liệt kê các hpa gõ lệnh
```
kubectl get hpa
```
- Để linh loạt và quy chuẩn, nên tạo ra HPA (HorizontalPodAutoscaler) từ cấu hình file yaml (Tham khảo HPA API ) , ví dụ: trong folder exams/2.replicaset

## Deployment
- Deployment quản lý một nhóm các Pod - các Pod được nhân bản, nó tự động thay thế các Pod bị lỗi, không phản hồi bằng pod mới nó tạo ra. Như vậy, deployment đảm bảo ứng dụng của bạn có một (hay nhiều) Pod để phục vụ các yêu cầu.

- Deployment sử dụng mẫu Pod (Pod template - chứa định nghĩa / thiết lập về Pod) để tạo các Pod (các nhân bản replica), khi template này thay đổi, các Pod mới sẽ được tạo để thay thế Pod cũ ngay lập tức.
