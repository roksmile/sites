# csi-driver-nfs Helm 차트를 이용하여 disconnected 환경에서 설치하기
> 준비사항
> 1. helm 명령어
> 2. podman 명령어
> 3. 인터넷이 되는 서버
> 4. 로컬 레지스트리 서버(ocp-registry.kscada.kdneri.com:5000)
> 5. NFS Server 주소: 10.60.1.28
> 6. NFS 마운트 포인트: /data/nfs-csi

## 1. helm repository 추가 
``` helm repo add csi-driver-nfs https://raw.githubusercontent.com/kubernetes-csi/csi-driver-nfs/master/charts ```

## 2. helm chart 다운로드 
```
helm pull csi-river-nfs/csi-driver-nfs
csi-driver-nfs-4.12.1.tgz
```

## 3. 컨테이너 이미지 다운로드 
```
podman pull registry.k8s.io/sig-storage/nfsplugin:v4.12.1
podman pull registry.k8s.io/sig-storage/csi-provisioner:v5.3.0
podman pull registry.k8s.io/sig-storage/csi-resizer:v1.14.0
podman pull registry.k8s.io/sig-storage/csi-snapshotter:v8.3.0
podman pull registry.k8s.io/sig-storage/livenessprobe:v2.17.0
podman pull registry.k8s.io/sig-storage/csi-node-driver-registrar:v2.15.0
podman pull registry.k8s.io/sig-storage/snapshot-controller:v8.3.0
```

## 4.로컬에 있는 이미지를 파일로 저장
```
podman save -o nfsplugin-v4.12.1-image.tar registry.k8s.io/sig-storage/nfsplugin:v4.12.1
podman save -o csi-provisioner-v5.3.0-image.tar registry.k8s.io/sig-storage/csi-provisioner:v5.3.0
...
podman save -o sanpshot-controller-v8.3.0-image.tar registry.k8s.io/sig-storage/snapshot-controller:v8.3.0
```

## 5.helm chart 파일(csi-driver-nfs-4.12.1.tgz)과 파일로 저장한 컨테이너 이미지를 폐쇄망 환경으로 복사한다.

## 6.컨테이너 이미지를 podman을 이용하여 로컬로 저장한다.
```
podman load -i nfsplugin-v4.12.1-image.tar
podman load -i csi-provisioner-v5.3.0-image.tar
...
podman load -i snapshot-controller-v8.3.0-image.tar
```

## 7.컨테이너 이미지를 로컬 레지스트리 서버 주소로 tagging 한다.
```
podman tag registry.k8s.io/sig-storage/nfsplugin:v4.12.1 ocp-registry.kscada.kdneri.com:5000/sig-storage/nfsplugin:v4.12.1
podman tag registry.k8s.io/sig-storage/csi-provisioner:v5.3.0 ocp-registry.kscada.kdneri.com:5000/sig-storage/csi-provisioner:v5.3.0
...
podman tag podman pull registry.k8s.io/sig-storage/snapshot-controller:v8.3.0 ocp-registry.kscada.kdneri.com:5000/sig-storage/snapshot-controller:v8.3.0
```

## 8.tagging한 이미지를 로컬 레지스트리 서버로 업로드 한다.
```
podman push ocp-registry.kscada.kdneri.com:5000/sig-storage/nfsplugin:v4.12.1
podman push ocp-registry.kscada.kdneri.com:5000/sig-storage/csi-provisioner:v5.3.0
...
podman push ocp-registry.kscada.kdneri.com:5000/sig-storage/snapshot-controller:v8.3.0
```

## 9.환경설정 파일(custom-values.yaml)을 생성한다.
```
storageClasses:
  - name: nfs-delete
    annotations:
      storageclass.kubernetes.io/is-default-class: "true"
    parameters:
      server: 10.60.1.28
      share: /data/nfs-csi
    reclaimPolicy: Delete
    volumeBindingMode: Immediate
    mountOptions:
      - nfsvers=4.1
  - name: nfs-retain
    parameters:
      server: 10.60.1.28
      share: /data/nfs-csi
    reclaimPolicy: Retain
    volumeBindingMode: Immediate
    mountOptions:
      - nfsvers=4.1
```

## 10.helm 명령어를 이용하여 설치 한다.
```
helm install csi-driver-nfs ./csi-driver-nfs-4.12.1.tgz \
--set image.baseRepo=ocp-registry.kscada.kdneri.com:5000 \
--namespace kube-system --version 4.12.0 \
--set image.nfs.repository=ocp-registry.kscada.kdneri.com:5000/sig-storage/nfsplugin \
--set image.csiProvisioner.repository=ocp-registry.kscada.kdneri.com:5000/sig-storage/csi-provisioner \
--set image.csiResizer.repository=ocp-registry.kscada.kdneri.com:5000/sig-storage/csi-resizer \
--set image.csiSnapshotter.repository=ocp-registry.kscada.kdneri.com:5000/sig-storage/csi-snapshotter \
--set image.livenessProbe.repository=ocp-registry.kscada.kdneri.com:5000/sig-storage/livenessprobe \
--set image.nodeDriverRegistrar.repository=ocp-registry.kscada.kdneri.com:5000/sig-storage/csi-node-driver-registrar \
--set image.externalSnapshotter.repository=ocp-registry.kscada.kdneri.com:5000/sig-storage/snapshot-controller \
-f custom-values.yaml
```

## 11.openshift에서 확인한다.
