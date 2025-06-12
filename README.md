# Побудова високо доступної системи з використанням контейнеризації та оркестрації

Курсовий проєкт студента групи ІТПА-11 **Дорожовця В. В.**.
Репозиторій містить вихідний код та конфігураційні файли для розгортання масштабованого веб-застосунку на Flask у кластері Kubernetes.

## 🎯 Мета проєкту

Продемонструвати практичну реалізацію високо доступної інформаційної системи за допомогою Kubernetes. Основна увага приділяється налаштуванню кластера, розгортанню сервісу та забезпеченню його стійкості до збоїв та масштабованості.

## 🛠️ Технології та інструменти

* **Оркестрація:** Kubernetes 
* **Контейнеризація:** Docker, containerd 
* **ОС:** Ubuntu Server 24.04.2 LTS 
* **Веб-фреймворк:** Flask 
* **Мережевий плагін:** Flannel 
* **Моніторинг для HPA:** Metrics Server 
* **Віртуалізація:** VirtualBox 

## 📁 Структура репозиторію
- k8s/                  - Конфігураційні файли Kubernetes
    - deployment.yaml   - Розгортання Flask-додатку з лімітами ресурсів
    - service.yaml      - Сервіс NodePort для доступу до додатку
    - pdb.yaml          - PodDisruptionBudget для високої доступності
- my_app/               - Вихідний код веб-застосунку
    - app.py            - Простий Flask-додаток
    - requirements.txt  - Залежності Python
    - Dockerfile        - Файл для створення Docker-образу
- README.md             - Документація проєкту

## 🚀 Інструкція з розгортання

### Крок 1: Підготовка інфраструктури

1.  Створіть три віртуальні машини (1 `master`, 2 `slave`) на VirtualBox з такими параметрами:
    * ОС: Ubuntu Server 24.04.2 LTS 
    * RAM: 2 ГБ 
    * CPU: 2 ядра 
    * Диск: 25 ГБ 
    * Мережа: Bridged Adapter 
2.  Налаштуйте статичні IP-адреси та `/etc/hosts` на кожному вузлі.
3.  Вимкніть swap на всіх вузлах.
4.  Налаштуйте необхідні порти брандмауера `ufw` для master та slave вузлів.

### Крок 2: Встановлення Kubernetes

1.  На всіх вузлах встановіть `docker-ce`, `containerd`, `kubelet`, `kubeadm`, `kubectl`.
2.  Налаштуйте `containerd` для використання `SystemdCgroup`.
3.  Налаштуйте параметри ядра для Kubernetes (`net.bridge.bridge-nf-call-iptables`, `net.ipv4.ip_forward`).

### Крок 3: Ініціалізація кластера

1.  На **master** вузлі ініціалізуйте кластер:
    ```bash
    sudo kubeadm init --apiserver-advertise-address=<IP_МАСТЕРА> --pod-network-cidr=10.244.0.0/16
    ```
2.  Налаштуйте `kubectl` для поточного користувача.
3.  Встановіть мережевий плагін Flannel:
    ```bash
    kubectl apply -f [https://github.com/flannel-io/flannel/releases/latest/download/kube-flannel.yml](https://github.com/flannel-io/flannel/releases/latest/download/kube-flannel.yml)
    ```
4.  Згенеруйте токен для приєднання робочих вузлів та підключіть **slave** вузли до кластера за допомогою отриманої команди `kubeadm join`.

### Крок 4: Розгортання застосунку

1.  **Створення Docker-образу**. На **master** вузлі перейдіть до папки `my_app`.
    ```bash
    # Створення образу
    docker build -t my-flask-app:latest . 

    # Вхід у Docker Hub
    docker login

    # Тегування та завантаження образу
    docker tag my-flask-app:latest volodymyrdorozhovets/coursework:latest 
    docker push volodymyrdorozhovets/coursework:latest 
    ```
2.  **Деплой в Kubernetes**. Перейдіть до папки `k8s` та застосуйте конфігурації:
    ```bash
    kubectl apply -f deployment.yaml 
    kubectl apply -f service.yaml 
    ```
3.  Перевірте статус:
    ```bash
    kubectl get pods -o wide
    kubectl get services
    ```
    Додаток буде доступний за адресою `http://<IP_ВУЗЛА>:30007`.

### Крок 5: Налаштування високої доступності та масштабування

1.  **Horizontal Pod Autoscaler (HPA)**.
    * Встановіть Metrics Server:
        ```bash
        kubectl apply -f [https://github.com/kubernetes-sigs/metrics-server/releases/latest/download/components.yaml](https://github.com/kubernetes-sigs/metrics-server/releases/latest/download/components.yaml)
        ```
    * Внесіть зміни в `deployment` Metrics Server для роботи з `kubelet-insecure-tls`.
    * Налаштуйте автоскейлінг для вашого `deployment`:
        ```bash
        kubectl autoscale deployment flask-app --cpu-percent=50 --min=3 --max=10 
        ```
2.  **PodDisruptionBudget (PDB)**.
    * Застосуйте PDB для гарантії мінімальної кількості доступних подів:
        ```bash
        kubectl apply -f pdb.yaml 
        ```

## ✅ Результат

В результаті виконання цих кроків розгорнуто високо доступну та масштабовану систему на базі Kubernetes. Система автоматично масштабується залежно від навантаження на CPU  та стійка до планових відключень вузлів завдяки PDB.
