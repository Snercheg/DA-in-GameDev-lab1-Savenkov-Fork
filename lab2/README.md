# АНАЛИЗ ДАННЫХ И ИСКУССТВЕННЫЙ ИНТЕЛЛЕКТ [in GameDev]
Отчет по лабораторной работе #2 выполнил:
- Савенков Александр Александрович
- ХПИ31
Отметка о выполнении заданий:

| Задание | Выполнение | Баллы |
| ------ | ------ | ------ |
| Задание 1 | * | 60 |
| Задание 2 | * | 20 |
| Задание 3 | * | 20 |

знак "*" - задание выполнено; знак "#" - задание не выполнено;

Работу проверили:
- к.т.н., доцент Денисов Д.В.
- к.э.н., доцент Панов М.А.
- ст. преп., Фадеев В.О.

[![N|Solid](https://cldup.com/dTxpPi9lDf.thumb.png)](https://nodesource.com/products/nsolid)

[![Build Status](https://travis-ci.org/joemccann/dillinger.svg?branch=master)](https://travis-ci.org/joemccann/dillinger)

Структура отчета

- Данные о работе: название работы, фио, группа, выполненные задания.
- Цель работы.
- Задание 1.
- Код реализации выполнения задания. Визуализация результатов выполнения (если применимо).
- Задание 2.
- Код реализации выполнения задания. Визуализация результатов выполнения (если применимо).
- Задание 3.
- Код реализации выполнения задания. Визуализация результатов выполнения (если применимо).
- Выводы.
- ✨Magic ✨

## Цель работы: познакомится с программными средствами для организации передачи данных между инструментами google, Python, Unity


## Задание 1
Ход работы:

- Создал проект на console.google.com для реализации связи Google Sheets, Python и Unity. Создал сервисный аккаунт и подключил API

- Реализовал запись данных из Python-скрипта в Google Sheets. Данные описывают изменения темпа инфляции.

```py
import gspread as g
import numpy as np

gc = g.service_account(filename="unitydatascience-364706-1c1ce20c7b07.json")
sh = gc.open("UnitySheets").worksheet('Лист1')
price = np.random.randint(2000, 10000, 11)

for i in range(1, 11):
    inf = ((price[i] - price[i - 1]) / price[i-1]) * 100

    sh.update(('A' + str(i)), str(i))
    sh.update(('B' + str(i)), str(price[i]))
    sh.update(('C' + str(i)), f"{inf:.3f}".replace(".", ","))
```

```csharp
using System.Collections;
using System.Collections.Generic;
using UnityEngine;
using UnityEngine.Networking;
using SimpleJSON;

public class NewBehaviourScript : MonoBehaviour
{
    public int column;
    public int firstRow = 1;
    public int rowCount = 1;

    public string url;
    public float lessValue;
    public float greaterValue;

    public AudioClip goodSpeak;
    public AudioClip normalSpeak;
    public AudioClip badSpeak;

    private AudioSource selectAudio;
    private Dictionary<string, float> dataSet = new Dictionary<string, float>();
    private bool statusStart = false;
    private int i = 1;

    // Start is called before the first frame update
    void Start()
    {
        i = firstRow;
        StartCoroutine(GoogleSheets());
    }

    // Update is called once per frame
    void Update()
    {
        if (i != (dataSet.Count + firstRow) && dataSet["Mon_" + i.ToString()] <= lessValue && statusStart == false)
        {
            StartCoroutine(PlaySelectAudioGood());
            Debug.Log(dataSet["Mon_" + i.ToString()]);
        }

        if (i != (dataSet.Count + firstRow) && dataSet["Mon_" + i.ToString()] > lessValue && dataSet["Mon_" + i.ToString()]
            < greaterValue && statusStart == false)
        {
            StartCoroutine(PlaySelectAudioNormal());
            Debug.Log(dataSet["Mon_" + i.ToString()]);
        }

        if (i != (dataSet.Count + firstRow) &&dataSet["Mon_" + i.ToString()] >= greaterValue && statusStart == false)
        {
            StartCoroutine(PlaySelectAudioBad());
            Debug.Log(dataSet["Mon_" + i.ToString()]);
        }
    }

    IEnumerator GoogleSheets() {
        UnityWebRequest currentResp = UnityWebRequest.Get("https://sheets.googleapis.com/v4/spreadsheets/1zbYNBbjBTkQNrmBpAyAdHjnM35UFv3QVf_PrJ52ie5s"));
        yield return currentResp.SendWebRequest();

        string rawResp = currentResp.downloadHandler.text;
        var rawJson = JSON.Parse(rawResp);

        for (int j = firstRow; j < rawJson["values"].Count && ((j <= rowCount + firstRow) || rowCount == 0); j++)
        {
            var parseJson = JSON.Parse(rawJson["values"][j].ToString());
            var selectRow = parseJson;
            dataSet.Add("Mon_" + j.ToString(), float.Parse(selectRow[column - 1]));
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

- Реализовал запись на вторую страницу в таблице(Лист2) набора данных, полученных с помощью линейной регрессии из 1 лаб. работы.

```python
from statistics import mode
import matplotlib.pyplot as plt
import gspread as g
import numpy as np

x = [3,21,22,34,54,34,55,67,89,99]
x = np.array(x)
y = [2,22,24,65,79,82,55,130,150,199]
y = np.array(y)

gc = g.service_account(filename='unitydatascience-364706-1c1ce20c7b07.json')
sh = gc.open("UnitySheets").worksheet('Лист2')

def model(w, b, x):
    return w*x + b

def lossFun(w, b, x, y):
    num = len(x)
    predic = model(w, b, x)
    return (0.5/num) * (np.square(predic-y)).sum()

def optimization(lr, w, b, x, y):
    num = len(x)
    predic = model(w, b, x)
    dw = (1.0/num) * ((predic - y) * x).sum()
    db = (1.0/num) * ((predic - y).sum())
    w = w - lr*dw
    b = b - lr*db
    return w, b

def iter(lr, w, b, x, y, times):
    for i in range(times):
        w, b = optimization(lr, w, b, x, y)
    return w,b  

lr = 0.000001
a_f = np.random.rand(1)
print(a_f)
b_f = np.random.rand(1)
print(b_f)

a = np.copy(a_f)
b = np.copy(b_f)
n_a = np.arange(1, 6) * 200
n_a = np.concatenate([[1], n_a])
a_loss = []

for i, n in enumerate(n_a):
    a, b = iter(lr, a, b, x, y, n)
    pred = model(a, b, x)
    loss = lossFun(a, b, x, y)
    a_loss.append(loss)

    print(a, b, loss)
    sh.update(('A' + str(i + 1)), str(n))
    sh.update(('B' + str(i + 1)), f"{a[0]:.3f}".replace('.',','))
    sh.update(('C' + str(i + 1)), f"{b[0]:.3f}".replace('.',','))
    sh.update(('D' + str(i + 1)), f"{loss:.3f}".replace('.',','))

```
## Задание 3


## Выводы
В ходе лабороторной работы я научился создавать наборы данных и передавать их в Google Sheets с помощью API. Научился воспроизводить звуковые файлы в Unity.

| Plugin | README |
| ------ | ------ |
| Dropbox | [plugins/dropbox/README.md][PlDb] |
| GitHub | [plugins/github/README.md][PlGh] |
| Google Drive | [plugins/googledrive/README.md][PlGd] |
| OneDrive | [plugins/onedrive/README.md][PlOd] |
| Medium | [plugins/medium/README.md][PlMe] |
| Google Analytics | [plugins/googleanalytics/README.md][PlGa] |

## Powered by

**BigDigital Team: Denisov | Fadeev | Panov**
