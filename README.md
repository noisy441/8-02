# Домашнее задание к занятию "`Что такое DevOps. СI/СD`" - `Дудин Сергей Васильевич`


### Инструкция по выполнению домашнего задания

   1. Сделайте `fork` данного репозитория к себе в Github и переименуйте его по названию или номеру занятия, например, https://github.com/имя-вашего-репозитория/git-hw или  https://github.com/имя-вашего-репозитория/7-1-ansible-hw).
   2. Выполните клонирование данного репозитория к себе на ПК с помощью команды `git clone`.
   3. Выполните домашнее задание и заполните у себя локально этот файл README.md:
      - впишите вверху название занятия и вашу фамилию и имя
      - в каждом задании добавьте решение в требуемом виде (текст/код/скриншоты/ссылка)
      - для корректного добавления скриншотов воспользуйтесь [инструкцией "Как вставить скриншот в шаблон с решением](https://github.com/netology-code/sys-pattern-homework/blob/main/screen-instruction.md)
      - при оформлении используйте возможности языка разметки md (коротко об этом можно посмотреть в [инструкции  по MarkDown](https://github.com/netology-code/sys-pattern-homework/blob/main/md-instruction.md))
   4. После завершения работы над домашним заданием сделайте коммит (`git commit -m "comment"`) и отправьте его на Github (`git push origin`);
   5. Для проверки домашнего задания преподавателем в личном кабинете прикрепите и отправьте ссылку на решение в виде md-файла в вашем Github.
   6. Любые вопросы по выполнению заданий спрашивайте в чате учебной группы и/или в разделе “Вопросы по заданию” в личном кабинете.
   
Желаем успехов в выполнении домашнего задания!
   
### Дополнительные материалы, которые могут быть полезны для выполнения задания

1. [Руководство по оформлению Markdown файлов](https://gist.github.com/Jekins/2bf2d0638163f1294637#Code)

---

### Задание 1

**Что нужно сделать:**

1. Установите себе jenkins по инструкции из лекции или любым другим способом из официальной документации. Использовать Docker в этом задании нежелательно.
2. Установите на машину с jenkins [golang](https://golang.org/doc/install).
3. Используя свой аккаунт на GitHub, сделайте себе форк [репозитория](https://github.com/netology-code/sdvps-materials.git). В этом же репозитории находится [дополнительный материал для выполнения ДЗ](https://github.com/netology-code/sdvps-materials/blob/main/CICD/8.2-hw.md).
3. Создайте в jenkins Freestyle Project, подключите получившийся репозиторий к нему и произведите запуск тестов и сборку проекта ```go test .``` и  ```docker build .```.

В качестве ответа пришлите скриншоты с настройками проекта и результатами выполнения сборки.

### Решение 1

![Скриншот 1](https://github.com/noisy441/8-02/blob/main/img/img1.png)`

![Скриншот 2](https://github.com/noisy441/8-02/blob/main/img/img2.png)`

![Скриншот 3](https://github.com/noisy441/8-02/blob/main/img/img3.png)`

![Скриншот 4](https://github.com/noisy441/8-02/blob/main/img/img4.png)`

![Скриншот 5](https://github.com/noisy441/8-02/blob/main/img/img5.png)`

![Скриншот 6](https://github.com/noisy441/8-02/blob/main/img/img6.png)`

---

### Задание 2

**Что нужно сделать:**

1. Создайте новый проект pipeline.
2. Перепишите сборку из задания 1 на declarative в виде кода.

В качестве ответа пришлите скриншоты с настройками проекта и результатами выполнения сборки.

### Решение 2

```
pipeline {
    agent any
    stages {
        stage('Git') {
            steps {
                git branch: 'main', url: 'https://github.com/noisy441/8-02-cicd.git'
            }
        }
        stage('Test') {
            steps {
                sh '/usr/local/go/bin/go test .'
            }
        }
        stage('Build') {
            steps {
                sh 'docker build . -t 158.160.95.105:8082/hello-world:v$BUILD_NUMBER'
            }
        }
        stage('Push') {
            steps {
                sh '''
                    docker login 158.160.95.105:8082 -u admin -p admin
                    docker push 158.160.95.105:8082/hello-world:v$BUILD_NUMBER
                    docker logout
                '''
            }
        }
    }
}
```

![Скриншот 7](https://github.com/noisy441/8-02/blob/main/img/img7.png)`
---

### Задание 3

**Что нужно сделать:**

1. Установите на машину Nexus.
1. Создайте raw-hosted репозиторий.
1. Измените pipeline так, чтобы вместо Docker-образа собирался бинарный go-файл. Команду можно скопировать из Dockerfile.
1. Загрузите файл в репозиторий с помощью jenkins.

В качестве ответа пришлите скриншоты с настройками проекта и результатами выполнения сборки.

### Решение 3

```
pipeline {
    agent any

    environment {
        NEXUS_URL = 'http://158.160.95.105:8081/repository/my-rep/'
        NEXUS_USER = 'admin'
        NEXUS_PASS = 'admin'
    }

    stages {
        stage('Checkout') {
            steps {
                git branch: 'main', url: 'https://github.com/noisy441/8-02-cicd.git'
            }
        }

        stage('Build Binary') {
            steps {
                sh '/usr/local/go/bin/go env'
                sh 'CGO_ENABLED=0 GOOS=linux /usr/local/go/bin/go build -a -installsuffix nocgo -o app .'
            }
        }

        stage('Archive Binary') {
            steps {
                archiveArtifacts artifacts: 'app', fingerprint: true
            }
        }

        stage('Upload to Nexus') {
            steps {
                script {
                    def binaryFile = 'app'
                    
                    sh """
                    curl -v -u ${NEXUS_USER}:${NEXUS_PASS} --upload-file ${binaryFile} ${NEXUS_URL}${binaryFile}
                    """
                }
            }
        }
    }

    post {
        success {
            echo 'Build and upload successful!'
        }
        failure {
            echo 'Build or upload failed.'
        }
    }
}

```

![Скриншот 8](https://github.com/noisy441/8-02/blob/main/img/img8.png)`

![Скриншот 9](https://github.com/noisy441/8-02/blob/main/img/img9.png)`
---
## Дополнительные задания* (со звёздочкой)

Их выполнение необязательное и не влияет на получение зачёта по домашнему заданию. Можете их решить, если хотите лучше разобраться в материале.

---

### Задание 4*

Придумайте способ версионировать приложение, чтобы каждый следующий запуск сборки присваивал имени файла новую версию. Таким образом, в репозитории Nexus будет храниться история релизов.

Подсказка: используйте переменную BUILD_NUMBER.

В качестве ответа пришлите скриншоты с настройками проекта и результатами выполнения сборки.

---
