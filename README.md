# АНАЛИЗ ДАННЫХ И ИСКУССТВЕННЫЙ ИНТЕЛЛЕКТ [in GameDev]
### ЛАБОРАТОРНАЯ РАБОТА. СБОР, ОБРАБОТКА И ВИЗУАЛИЗАЦИЯ ТЕСТОВОГО НАБОРА ДАННЫХ

Отчет по лабораторной работе #2 выполнил
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
- Выводы
- Сноски

## Цель работы
### *Познакомиться с программными средствами для организции передачи данных между инструментами Google, Python и Unity*

## Задание 1
### *Реализовать совместную работу и передачу данных в связке Python – Google-Sheets – Unity.*

### 1. В сервисе Google Cloud были подключены необходимые API для работы с Google Sheets и Google Cloud, был создан API-ключ.

### 2. Была реализована запись данных из [Python-скрипта](https://github.com/krakyz/AD_LR1/blob/11119d9dce5f234aca05480fdb8668e65b8cde03/Task%201/main.py) в [Google Sheets](https://docs.google.com/spreadsheets/d/e/2PACX-1vTSQFc8K0YwUdlbgVRLlhKqBOEK4F94iu9LIe14el8L4Z2nmmoX6mhRj-waHR-DtPOXNzfus6IORPTJ/pubhtml?gid=0&single=true)
```py
import gspread
import numpy as np

gc = gspread.service_account(filename='avian-influence-365213-6502a69b35aa.json')
sh = gc.open("UnitySheets")
price = np.random.randint(2000, 10000, 11)
mon = list(range(1, 11))
i = 0
while i <= len(mon):
    i += 1
    if i == 0:
        continue
    else:
        tempInf = ((price[i - 1] - price[i - 2]) / price[i - 2]) * 100
        tempInf = str(tempInf)
        tempInf = tempInf.replace('.', ',')
        sh.sheet1.update(('A' + str(i)), str(i))
        sh.sheet1.update(('B' + str(i)), str(price[i - 1]))
        sh.sheet1.update(('C' + str(i)), str(tempInf))
        print(tempInf)
```
Данные описывают изменение темпа инфляции на протяжении 11 отсчётных периодов, с учётом стоимости игрового объекта в каждый период.
### 3. Создан новый [Unity-проект](https://github.com/krakyz/AD_LR1/tree/main/Task%201/UnityDataScience/Assets)¹, получающий данные из [Google Sheets](https://docs.google.com/spreadsheets/d/e/2PACX-1vTSQFc8K0YwUdlbgVRLlhKqBOEK4F94iu9LIe14el8L4Z2nmmoX6mhRj-waHR-DtPOXNzfus6IORPTJ/pubhtml?gid=0&single=true)

### 4. Написан [Unity-скрипт](https://github.com/krakyz/AD_LR1/blob/11119d9dce5f234aca05480fdb8668e65b8cde03/Task%201/UnityDataScience/Assets/Script/NewBehaviourScript.cs), воспроизводящий аудио-файл в зависимости от значения данных из таблицы [Google Sheets](https://docs.google.com/spreadsheets/d/e/2PACX-1vTSQFc8K0YwUdlbgVRLlhKqBOEK4F94iu9LIe14el8L4Z2nmmoX6mhRj-waHR-DtPOXNzfus6IORPTJ/pubhtml?gid=0&single=true).
```c#
using System.Collections;
using System.Collections.Generic;
using UnityEngine;
using UnityEngine.Networking;
using SimpleJSON;

public class NewBehaviourScript : MonoBehaviour
{
    public AudioClip goodSpeak;
    public AudioClip normalSpeak;
    public AudioClip badSpeak;
    private AudioSource selectAudio;
    private Dictionary<string,float> dataSet = new Dictionary<string, float>();
    private bool statusStart = false;
    private int i = 1;

    void Start()
    {
        StartCoroutine(GoogleSheets());
    }

    void Update()
    {
        if (dataSet["Mon_" + i.ToString()] <= 10 & statusStart == false & i != dataSet.Count)
        {
            StartCoroutine(PlaySelectAudioGood());
            Debug.Log(dataSet["Mon_" + i.ToString()]);
        }

        if (dataSet["Mon_" + i.ToString()] > 10 & dataSet["Mon_" + i.ToString()] < 100 & statusStart == false & i != dataSet.Count)
        {
            StartCoroutine(PlaySelectAudioNormal());
            Debug.Log(dataSet["Mon_" + i.ToString()]);
        }

        if (dataSet["Mon_" + i.ToString()] >= 100 & statusStart == false & i != dataSet.Count)
        {
            StartCoroutine(PlaySelectAudioBad());
            Debug.Log(dataSet["Mon_" + i.ToString()]);
        }
    }

    IEnumerator GoogleSheets()
    {
        UnityWebRequest curentResp = UnityWebRequest.Get("https://sheets.googleapis.com/v4/spreadsheets/1A_fsUi6OTVkIWjECjvr51pRtwI0kMSJLBPjakKz3Mao/values/Sheet1?key=AIzaSyDeTNuLdoeoI-eKkuz9VfHfsx8dD6X4u-Q");
        yield return curentResp.SendWebRequest();
        string rawResp = curentResp.downloadHandler.text;
        var rawJson = JSON.Parse(rawResp);
        foreach (var itemRawJson in rawJson["values"])
        {
            var parseJson = JSON.Parse(itemRawJson.ToString());
            var selectRow = parseJson[0].AsStringList;
            dataSet.Add(("Mon_" + selectRow[0]), float.Parse(selectRow[2]));
        }
    }

    IEnumerator PlaySelectAudioGood()
    {
        statusStart = true;
        selectAudio = GetComponent<AudioSource>();
        selectAudio.clip = goodSpeak;
        selectAudio.Play();
        yield return new WaitForSeconds(3);
        statusStart = false;
        i++;
    }
    IEnumerator PlaySelectAudioNormal()
    {
        statusStart = true;
        selectAudio = GetComponent<AudioSource>();
        selectAudio.clip = normalSpeak;
        selectAudio.Play();
        yield return new WaitForSeconds(3);
        statusStart = false;
        i++;
    }
    IEnumerator PlaySelectAudioBad()
    {
        statusStart = true;
        selectAudio = GetComponent<AudioSource>();
        selectAudio.clip = badSpeak;
        selectAudio.Play();
        yield return new WaitForSeconds(4);
        statusStart = false;
        i++;
    }
}
```

## Задание 2
### *Реализовать запись в Google-таблицу набора данных, полученных с помощью линейной регрессии из лабораторной работы № 1*

### 1. Был создан ещё один ключ для использования в скрипте

### 2. Код, используемый в лабораторный работе №1 был [изменен и дополнен](https://github.com/krakyz/AD_LR1/blob/main/Task%202/LR2_T2.py) для использования API

```py
import numpy as np
import matplotlib.pyplot as plt
import gspread


def model(a, b, x):
    return a * x + b


def loss_function(a, b, x, y):
    num = len(x)
    prediction = model(a, b, x)
    return (0.5 / num) * (np.square(prediction - y)).sum()


def optimize(a, b, x, y, Lr):
    num = len(x)
    prediction = model(a, b, x)
    da = (1.0 / num) * ((prediction - y) * x).sum()
    db = (1.0 / num) * (prediction - y).sum()
    a = a - Lr * da
    b = b - Lr * db
    return a, b


def iterate(a, b, x, y, times, Lr):
    for i in range(times):
        a, b = optimize(a, b, x, y, Lr)
    return a, b


gc = gspread.service_account(filename='avian-influence-365213-de6d57216284.json')
sh = gc.open('LR2_T2')

x = [3, 21, 22, 34, 54, 34, 55, 67, 89, 99]
x = np.array(x)
y = [2, 22, 24, 65, 79, 82, 55, 130, 150, 99]
y = np.array(y)
Lr = 0.000001

times = [1, 2, 3, 4, 5, 6, 10, 100, 1000, 10000]

for i in range(1, len(times) + 1):
    a, b = np.random.rand(1), np.random.rand(1)
    a, b = iterate(a, b, x, y, times[i - 1], Lr)
    prediction = model(a, b, x)

    sh.sheet1.update(('A' + str(i)), str(i))
    sh.sheet1.update(('B' + str(i)), str(a[0]))
    sh.sheet1.update(('C' + str(i)), str(b[0]))
    sh.sheet1.update(('D' + str(i)), str(loss_function(a, b, x, y)))
    print(loss_function(a, b, x, y))
```

## Выводы

В ходе работы я познакомился с программными средствами для организции передачи данных между инструментами Google, Python и Unity. Настроил импорт выходных данных Python-скрипты в таблицу Google Sheets и импорт из таблицы Google Sheets в Unity.

## Сноски

¹ На GitHub были загружены только пользовательские файлы — стандартные файлы, создаваемые автоматически при создании проекта, загружены не были.
