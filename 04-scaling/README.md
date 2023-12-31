# Домашнее задание к занятию «Микросервисы: масштабирование»

Вы работаете в крупной компании, которая строит систему на основе микросервисной архитектуры.
Вам как DevOps-специалисту необходимо выдвинуть предложение по организации инфраструктуры для разработки и эксплуатации.

## Задача 1: Кластеризация

Предложите решение для обеспечения развёртывания, запуска и управления приложениями.
Решение может состоять из одного или нескольких программных продуктов и должно описывать способы и принципы их взаимодействия.

Решение должно соответствовать следующим требованиям:
- поддержка контейнеров;
- обеспечивать обнаружение сервисов и маршрутизацию запросов;
- обеспечивать возможность горизонтального масштабирования;
- обеспечивать возможность автоматического масштабирования;
- обеспечивать явное разделение ресурсов, доступных извне и внутри системы;
- обеспечивать возможность конфигурировать приложения с помощью переменных среды, в том числе с возможностью безопасного хранения чувствительных данных таких как пароли, ключи доступа, ключи шифрования и т. п.

Обоснуйте свой выбор.

---

Рассмотрим популярные решения [Apache Mesos](https://mesos.apache.org/), [Red Hat Openshift](https://www.redhat.com/en/technologies/cloud-computing/openshift) и [Kubernetes](https://kubernetes.io).

| Требование                                                        | Apache Mesos   | OpenShift | Kubernetes |
|-------------------------------------------------------------------|----------------|-----------|------------|
| Поддержка контейнеров                                             | +              | +         | +          |
| Обнаружение сервисов и маршрутизация запросов                     | +              | +         | +          |
| Возможность горизонтального масштабирования                       | +              | +         | +          |
| Возможность автоматического масштабирования                       | +              | +         | +          |
| Явное разделение ресурсов доступных извне и внутри системы        | +              | +         | +          |
| Возможность конфигурировать приложения с помощью переменных среды | +              | +         | +          |

Из таблицы видно, что все решения удовлетворяют поставленным требованиям.
В качестве решения я бы рекомендовал Kubernetes, так как дополнительными плюсами будут:
- большая распространённость технологии - легче найти специалиста по обслуживанию; 
- большое комьюнити;
- поддержка со стороны большинства облачных платформ - позволит при необходимости быстро выполнить миграцию;
- высокая стабильность работы.