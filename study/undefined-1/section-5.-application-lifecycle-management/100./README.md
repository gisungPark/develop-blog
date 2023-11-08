# 100. 쿠버네티스 환경변수 설정

이전 강의에서 포드 정의 파일에서 환경 변수를 설정하는 법을 학습했다.

그러나, 포드 정의 파일이 많은 경우 저장된 환경 관리가 어려워 진다. 때문에 포드 정의 파일에서 이 정보를 가져와 configMap을 통해 관리할 수 있다. configMap은 key-value 구조의 데이터를 전달하는데 사용된다.

포드가 생성되면 configMap에서 key-value 구조의 데이터를 읽어와 환경 정보를 설정할 수 있다.



## ConfigMap 구성의 2가지 단계

1. configMap을 생성한다.
2. 생성된 configMap을 Pod에 주입한다.

## ConfigMap을 만드는 방법

1. 선언적 방법

> kubectl create configmap
>
> &#x20;   \<config-name>  --from-literal=\<key>=\<value>
>
> \
> kubectl create configmap \\
>
> &#x20;   app-config --from-literal=APP\_COLOR=blue  \\\
> &#x20;                 \--from-literal=APP\_MOD=prod



2. 파일 생성법

> kubectl create configmap
>
> &#x20;   \<config-name>  --from-file=\<path-to-file>
>
> \
> kubectl create configmap \\
>
> &#x20;   app-config --from-file=app\_config.properties



### ConfigMap 구성

config-map.yaml

> apiVersion: v1
>
> kind: ConfigMap
>
> metadata:\
> &#x20;   name: app-config
>
> \
> data:
>
> &#x20;   APP\_COLOR: blue
>
> &#x20;   APP\_MODE: prod\
> \
>

## configMap을 Pod에 주입하는 방법



