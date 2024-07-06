# flux-multibranch-multienv-example

Цей репозиторій є демонстрацією того, як можна за допомогою Flux синхронізувати кілька гілок
одного репозиторію з кількома неймспейсами в Kubernetes.

Демонстраційний додаток, описаний за допомогою маніфестів Kubernetes, знаходиться в репозиторії [yevgen-grytsay/flux-multibranch-multienv-app](https://github.com/yevgen-grytsay/flux-multibranch-multienv-app). Цей репозиторій має дві гілки: `main` і `dev`. Маніфести з гілки `main` встановлюються у неймспейс `prod`, а маніфести з гілки `dev` -- у неймспейс `dev`.

## Setup

```sh
flux check --pre
flux install

read -s GITHUB_TOKEN
export GITHUB_TOKEN

# PAT with permissions "Contents: Read and Write"
flux bootstrap github \
 --owner=yevgen-grytsay \
 --repository=flux-multibranch-multienv-example \
 --personal \
 --path=cluster \
 --token-auth
```

Після ініціалізації Flux, коли будуть створені неймспейси `prod` і `dev`, додати параметри доступу

```sh
kubectl create secret generic hello-world \
    -n dev \
    --from-literal=username=yevgen-grytsay \
    --from-literal=password="${GITHUB_TOKEN}"

kubectl create secret generic hello-world \
    -n prod \
    --from-literal=username=yevgen-grytsay \
    --from-literal=password="${GITHUB_TOKEN}"
```

Так виглядають ресурси `GitRepository` в Kubernetes:

```
# kubectl get gitrepo -A
NAMESPACE     NAME               URL                                                                       AGE   READY   STATUS
dev           hello-world-dev    https://github.com/yevgen-grytsay/flux-multi-branch-multi-env-app         70s   True    stored artifact for revision 'dev@sha1:b5ae33b162eb5041dfba3aaad39779a99e70bddf'
flux-system   flux-system        https://github.com/yevgen-grytsay/flux-multibranch-multienv-example.git   36m   True    stored artifact for revision 'main@sha1:2fb28a043a224f2cdee764523d89fd291215658e'
prod          hello-world-prod   https://github.com/yevgen-grytsay/flux-multi-branch-multi-env-app         10s   True    stored artifact for revision 'main@sha1:b5ae33b162eb5041dfba3aaad39779a99e70bddf'
```

## Приватний репозиторій

Параметри доступу до репозиторію з додатком, якщо він приватний, можна зберігати в сікреті. Сікрет має бути в одному неймспейсі з ресурсом `GitRepository`. Детальніше про це в [документації](https://fluxcd.io/flux/components/source/gitrepositories/#secret-reference).

```sh
apiVersion: source.toolkit.fluxcd.io/v1
kind: GitRepository
metadata:
  name: hello-world-dev
spec:
  url: https://github.com/yevgen-grytsay/flux-multi-branch-multi-env-app
  secretRef:
    name: hello-world
```
