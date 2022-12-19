# АНАЛИЗ ДАННЫХ И ИСКУССТВЕННЫЙ ИНТЕЛЛЕКТ [in GameDev]
### ЛАБОРАТОРНАЯ РАБОТА. РАЗРАБОТКА СИСТЕМЫ МАШИННОГО ОБУЧЕНИЯ

Отчет по лабораторной работе #3 выполнил
- Сурков Степан Васильевич
- РИ-210915
### Отметка о выполнении заданий (заполняется студентом):
| Задание | Выполнение | Баллы |
| ------ | ------ | ------ |
| Задание 1 | * | 60 |
| Задание 2 | * | 20 |
| Задание 3 | # | 20 |

знак "*" - задание выполнено; знак "#" - задание не выполнено;

### Работу проверили:
- к. т. н., доцент Денисов Д.В.
- к. э. н., доцент Панов М.А.
- ст. преп., Фадеев В.О.

### Структура отчета

- Данные о работе: название работы, ФИО, группа, выполненные задания
- Цель работы
- Задание 1
- Код реализации выполнения задания. Визуализация результатов выполнения (если применимо)
- Задание 2
- Код реализации выполнения задания. Визуализация результатов выполнения (если применимо)
- Задание 3
- Код реализации выполнения задания. Визуализация результатов выполнения (если применимо)
- Выводы
- Сноски

## Цель работы
### *познакомиться с программными средствами для создания системы машинного обучения и ее интеграции в Unity.*

## Задание 1
### *Реализовать систему машинного обучения в связке Python - Google-Sheets – Unity.*

### 1. Создан пустой 3D проект в Unity
![image](https://user-images.githubusercontent.com/66885212/208491945-60894771-10bc-45f6-9253-d47fdeae72eb.png)

### 2. К Unity подключены пакеты ML Agent и ML Agent Extensions
![image](https://user-images.githubusercontent.com/66885212/208492676-48a3f8ff-47a8-48f3-9238-7ade571cc865.png)


### 3. Установлены библиотеки ML Agent и Torch
`pip install mlagents==28.0`

![image](https://user-images.githubusercontent.com/66885212/208494346-30261d41-63e4-4289-b86f-478e3334afe7.png)

`conda install pytorch torchvision torchaudio cpuonly -c pytorch`

![image](https://user-images.githubusercontent.com/66885212/208498886-c618eae6-b590-4804-8a30-24c17f63d330.png)

### 4. Добавлены объекты: Plane, Sphere (RollerAgent), Cube (Target). Создан скрипт для Roller Agent.
![image](https://user-images.githubusercontent.com/66885212/208502115-721a6435-5ca3-4e6d-8e53-0f27016a3563.png)

### 5. Создан скрипт RollerAgent.
```c#
using System.Collections;
using System.Collections.Generic;
using UnityEngine;
using Unity.MLAgents;
using Unity.MLAgents.Sensors;
using Unity.MLAgents.Actuators;

public class RollerAgent : Agent
{
    Rigidbody rBody;

    void Start()
    {
        rBody = GetComponent<Rigidbody>();
    }

    public Transform Target;
    public override void OnEpisodeBegin()
    {
        if (this.transform.localPosition.y < 0)
        {
            this.rBody.angularVelocity = Vector3.zero;
            this.rBody.velocity = Vector3.zero;
            this.transform.localPosition = new Vector3(0, 0.5f, 0);
        }
        Target.localPosition = new Vector3(Random.value * 8-4, 0.5f, Random.value * 8-4);
    }

    public override void CollectObservations(VectorSensor sensor)
    {
        sensor.AddObservation(Target.localPosition);
        sensor.AddObservation(this.transform.localPosition);
        sensor.AddObservation(rBody.velocity.x);
        sensor.AddObservation(rBody.velocity.z);
    }

    public float forceMultiplier = 10;
    public override void OnActionReceived(ActionBuffers actionBuffers)
    {
        Vector3 controlSignal = Vector3.zero;
        controlSignal.x = actionBuffers.ContinuousActions[0];
        controlSignal.z = actionBuffers.ContinuousActions[1];
        rBody.AddForce(controlSignal * forceMultiplier);
        float distanceToTarget = Vector3.Distance(this.transform.localPosition, Target.localPosition);
        if (distanceToTarget < 1.42f)
        {
            SetReward(1.0f);
            EndEpisode();
        }
        else if (this.transform.localPosition.y < 0)
            EndEpisode();
    }
}
```

### 6. Добавлена цель для скрипта RollerAgent. Установлены и настроены скрипты DecisionRequester и Behavior Parameters.
![image](https://user-images.githubusercontent.com/66885212/208502873-9ea7d2b7-7fdd-4e7f-b4a8-c4f716812d66.png)

### 7. В директории MLAgent создан и отредактирован файл rollerball_config.yaml.
```yaml
behaviors:
  RollerBall:
    trainer_type: ppo
    hyperparameters:
      batch_size: 10
      buffer_size: 100
      learning_rate: 3.0e-4
      beta: 5.0e-4
      epsilon: 0.2
      lambd: 0.99
      num_epoch: 3
      learning_rate_schedule: linear
    network_settings:
      normalize: false
      hidden_units: 128
      num_layers: 2
    reward_signals:
      extrinsic:
        gamma: 0.99
        strength: 1.0
    max_steps: 500000
    time_horizon: 64
    summary_freq: 10000
```

### 8. Запущен MLAgent
![image](https://user-images.githubusercontent.com/66885212/208516800-696da8a8-ef31-46c6-8dc5-a030ad1ed9e5.png)
![image](https://user-images.githubusercontent.com/66885212/208516945-5996b97a-3ae2-4112-86be-0cd441d11b89.png)

### 9. Запущен MLAgent с девятью моделями
![image](https://user-images.githubusercontent.com/66885212/208522090-0ace70f9-6cc5-49d4-bdbc-c3a1355f26ba.png)

### 10. Запущен MLAgent с тридцатью шестью моделями
![image](https://user-images.githubusercontent.com/66885212/208523526-608502bf-f995-4502-833d-5f7b0d00b379.png)

### 11. Подключаем результаты обучения к скрипту Behavior Parameters. Обученный RollerAgent быстро находит цель и совершает меньше ошибок.

Behavior Parameters – обученная модель

![ezgif com-gif-maker](https://user-images.githubusercontent.com/66885212/208525268-b00d9c39-f16a-420e-8f2c-bf2ef41076b8.gif)

Behavior Parameters – необученная модель

![ezgif com-gif-maker (1)](https://user-images.githubusercontent.com/66885212/208525848-ff22a6fa-c0ad-464c-a77b-e3786ba1f27d.gif)

## Выводы



## Сноски

¹ На GitHub были загружены только пользовательские файлы — стандартные файлы, создаваемые автоматически при создании проекта, загружены не были.
