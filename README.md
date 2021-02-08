# AleksandrZooLZakharov_platform
AleksandrZooLZakharov Platform repository

ДЗ 01 Настройка локального окружения. Запуск окружения. Запуск первого контейнера. Работа с kubectl.
1. Поды неймспейса kube-system используют priority-class вида system-critical. Таким образом, кластер будет всегда перезапускать такие поды. Подробнее тут. https://kubernetes.io/docs/tasks/administer-cluster/guaranteed-scheduling-critical-addon-pods/ При этом coredns & kube-proxy управляются соответственно replicaset & daemonset, a остальные поды - самой нодой.
2.* Причина, по которой Под с frontend  находился в статусе error - микросервису не хватало информации о необходимых для его работы переменных окружения. После добавления переменных окружения из манифеста для фронтенда - всё "заработало".

ДЗ 02 Kubernetes controllers: ReplicaSet, Deployment, DaemonSet.
1. Обновление ReplicaSet не повлекло обновление ВЕРСИЙ запущенных pod, потому что репликасет следит только за количеством запущенных подконтрольных (подходящих под селектор) подов, но не за их версиями.

2. Предложенные стратегии обновления реализуются добавлением в spec деплоймента раздела:
  strategy:
    rollingUpdate:
      maxSurge: %1
      maxUnavailable: %2
  И при значениях соответственно 3 и 0 реализуется "Аналог blue-green", а при значениях 0 и 1 реализуется "Reverse Rolling Update".

ДЗ 06 Шаблонизация манифестов. Helm и его аналоги (Jsonnet, Kustomize) 
1 задание со звёздочкой:
	* Установка Chartmuseum должна производиться следующим образом:
	helm upgrade --install chartmuseum stable/chartmuseum --wait  --namespace=chartmuseum  --version=2.13.2 --set env.open.DISABLE_API=false -f values.yaml
	!!!Важен параметр DISABLE_API, который по умолчанию установлен в true и запрещает обращаться к /api. 
	!!!Или нужно в values.yaml изменить парамент DISABLE_API на false.
	
	* После этого добавляем музей к списком репозиториев:
	helm repo add chartmuseum https://chartmuseum.35.233.79.169.nip.io --insecure-skip-tls-verify
	!!!Важен параметр --insecure-skip-tls-verify (из-за самоподписанности TLS)

	* Для простоты\удобства использовал плагин хельма
	helm plugin install https://github.com/chartmuseum/helm-push.git

	* И уже спокойно пушим нужные чарты в репу chartmuseum (архивом или папкой):
	helm push --insecure %chartname.tgz%|%chartdir/% chartmuseum
	!!!Важен параметр --insecure (из-за самоподписанности TLS)

	* После этого чартом из этой репы можно пользоваться как и любым другим, вот только пока сертификат самоподписанный, придётся добавлять ключ --insecure-skip-tls-verify
	Например, в домашке я ставил Харбор из своего музея (предварительно спулив его из правильной репы с помощью helm pull)
	helm upgrade --install harbor chartmuseum/harbor --wait  --insecure-skip-tls-verify --namespace=harbor  --version=1.1.2  -f values.yaml
