# Баланс в играх - Идеальный баланс
Отчет по лабораторной работе #3 выполнил(а):
- Маврин Александр Андреевич
- НМТ-232513

| Задание | Выполнение | Баллы |
| ------ | ------ | ------ |
| Задание 1 | * | 60 |
| Задание 2 | * | 20 |
| Задание 3 | * | 20 |

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

## Цель работы
Разработать / изменить баланс сложностей для 10 уровней игры Dragon Picker

## Задание 1. Начало работы
### Ознакомиться с презентацией третьей лекции. Оценить структуру игры Dragon Picker. Скачать и ознакомиться с прототипом игры Dragon Picker на Unity. Запустите игровую сцену, проанализируйте движение дракона:
### Какие переменные влияют на движение дракона на сцене? Укажите эти переменные
— На движение дракона влияют параметры: скорости и рандомное значение, отвечающее за то, будет ли дракон менять траекторию в определенный момент времени

### Какие переменные влияют на сложность игры на сцене? Укажите эти переменные
— На сложность игры влияют параметры: скорость дракона, частота появления яйца, скорость падения яйца, количество щитов, частота изменения направления движения дракона и размер щита

## Задание 2
### Начало работы с шаблоном google-таблицы для балансировки игры 
### Отметьте, как может быть использован шаблон таблицы для визуализации изменения уровней сложности в игре Dragon Picker?
— Шаблон может быть использован для того, чтобы отследить как меняются переменные "баланса" в зависимости от каждого уровня. Так же для удобства их можно визуализировать графиками.

## Задание 3
### №1. Предложите вариант изменения найденных переменных для 10 уровней в игре. Визуализируйте изменение уровня сложности в таблице. 
```Python
import gspread
import numpy as np

gc = gspread.service_account(filename="dragon-b4afb65f2b89.json")
sh = gc.open("dragon")

def update_indexes():
    for i in range(1, 11):
        sh.sheet1.update_acell(f"A{i+1}", str(i))

def update_dragon_speed(percent: str):
    current_speed = float(sh.sheet1.acell("B2").value)
    multiplier = 1 + float(percent) / 100
    for i in range(3, 12):
        current_speed *= multiplier
        formatted_speed = f"{int(current_speed)},{int((current_speed - int(current_speed)) * 100)}"
        sh.sheet1.update_acell(f"B{i}", formatted_speed)

def update_time_egg_drop(percent: str):
    tail = 1 - int(percent) / 100
    current_time = float(sh.sheet1.acell("C2").value)
    for i in range(3, 12):
        current_time *= tail
        formatted_time = f"{int(current_time)},{int((current_time - int(current_time)) * 100)}"
        sh.sheet1.update_acell(f"C{i}", formatted_time)

def update_shields_count():
    current_count = int(sh.sheet1.acell("E2").value)
    repeat = 1
    for i in range(3, 12):
        sh.sheet1.update_acell(f"E{i}", str(current_count))
        repeat += 1
        if repeat == 3:
            current_count -= 1
            repeat = 0

if __name__ == "__main__":
    update_indexes()
    update_dragon_speed("20")
    update_time_egg_drop("10")
    update_shields_count()

```
## Таблица 
![image](https://github.com/user-attachments/assets/242b1a75-c09a-4af5-955f-286271876aad)
Ссылка на таблицу: [https://docs.google.com/spreadsheets/d/1GmpemND9ZYDorn7ckyxHTwsnB5gb_BmIGuvw1UWP2JA/edit?usp=sharing](https://docs.google.com/spreadsheets/d/1rtkJyDXb62VLfJXMdxZI8IFM6chAppjtJI02RCoCD0M/edit?gid=0#gid=0)

### №2 Создайте 10 сцен на Unity с изменяющимся уровнем сложности.
### №3 Решение в 80+ баллов должно визуализировать данные из google-таблицы, и с помощью Python передавать в проект Unity. В Python данные также должны быть визуализированы.
Скрипт компанента:
``` C#
using System;
using System.Collections;
using UnityEngine;
using UnityEngine.Networking;
using SimpleJSON;
using System.Globalization;

public class LoadSettings : MonoBehaviour
{
    [SerializeField] private int _sceneNumber;

    #region Private Fields
    private float _dragonSpeed;
    private float _timeEggDrop;
    private float _mass;
    private int _countShield;
    #endregion

    #region Getters
    public float DragonSpeed => _dragonSpeed;
    public float TimeEggDrop => _timeEggDrop;
    public float Mass => _mass;
    public int CountShield => _countShield;
    #endregion

    public event Action CoroutineIsStopped;

    void Awake()
    {
        StartCoroutine(GoogleSheets());
    }

    private float ConvertStrToFloat(string str)
    {
        if (string.IsNullOrWhiteSpace(str))
        {
            Debug.LogError("String is null or empty.");
            return 0f; // Значение по умолчанию
        }

        str = str.Replace(',', '.');
        if (!float.TryParse(str, NumberStyles.Float, CultureInfo.InvariantCulture, out var result))
        {
            Debug.LogError($"Failed to convert '{str}' to float.");
            return 0f; // Значение по умолчанию
        }

        return result;
    }

    IEnumerator GoogleSheets()
    {
        string url = "https://sheets.googleapis.com/v4/spreadsheets/1GmpemND9ZYDorn7ckyxHTwsnB5gb_BmIGuvw1UWP2JA/values/Лист1?key=AIzaSyBzuQEHb02mqr_SodKQpAWk5PL6YdZdXAc";
        UnityWebRequest curentResp = UnityWebRequest.Get(url);
        yield return curentResp.SendWebRequest();

        if (curentResp.result != UnityWebRequest.Result.Success)
        {
            Debug.LogError($"Error fetching Google Sheets data: {curentResp.error}");
            yield break; // Завершить выполнение корутины при ошибке
        }

        string rawResp = curentResp.downloadHandler.text;
        var rawJson = JSON.Parse(rawResp);

        var values = rawJson["values"];
        if (values == null)
        {
            Debug.LogError("JSON does not contain 'values'.");
            yield break;
        }

        bool isDataLoaded = false;

        foreach (var itemRawJson in values)
        {
            var parseJson = JSON.Parse(itemRawJson.ToString())[0];
            if (parseJson == null)
            {
                Debug.LogError("Invalid JSON structure.");
                continue;
            }

            string lvl = parseJson[0];
            if (int.TryParse(lvl, out var intLvl) && _sceneNumber == intLvl)
            {
                isDataLoaded = true;
                Debug.Log($"Data found for scene: {_sceneNumber}");

                _dragonSpeed = ConvertStrToFloat(parseJson[1]);
                _timeEggDrop = ConvertStrToFloat(parseJson[2]);
                _mass = ConvertStrToFloat(parseJson[3]);
                _countShield = int.TryParse(parseJson[4], out var shields) ? shields : 0;

                Debug.Log($"Dragon Speed: {_dragonSpeed}, Egg Drop Time: {_timeEggDrop}, Mass: {_mass}, Shields: {_countShield}");
                break;
            }
        }

        if (!isDataLoaded)
        {
            Debug.LogWarning($"No data found for scene number: {_sceneNumber}.");
        }

        CoroutineIsStopped?.Invoke(); // Уведомление о завершении
    }
}
```
Обработчик события
```C#
using System.Collections;
using UnityEngine;

public class EnemyDragon : MonoBehaviour
{
    public GameObject dragonEggPrefab;

    [SerializeField] private LoadSettings _settings;
    [SerializeField] private float leftRightDistance = 10f;
    [SerializeField] private float chanceDirection = 0.1f;

    private float _speed;
    private float _timeBetweenEggDrops;

    private void OnEnable()
    {
        // Подписываемся на событие обновления настроек
        _settings.CoroutineIsStopped += UpdateFields;
    }

    private void OnDisable()
    {
        // Отписываемся от события при отключении объекта
        _settings.CoroutineIsStopped -= UpdateFields;
    }

    private void Start()
    {
        // Первое сбрасывание яйца
        Invoke(nameof(DropEgg), 2f);
    }

    private void UpdateFields()
    {
        _speed = _settings.DragonSpeed;
        _timeBetweenEggDrops = _settings.TimeEggDrop;

        Debug.Log($"Dragon settings updated: Speed = {_speed}, Egg Drop Interval = {_timeBetweenEggDrops}");
    }

    private void DropEgg()
    {
        if (dragonEggPrefab == null)
        {
            Debug.LogError("Dragon egg prefab is not assigned!");
            return;
        }

        Vector3 eggSpawnPosition = transform.position + Vector3.up * 5f;
        Instantiate(dragonEggPrefab, eggSpawnPosition, Quaternion.identity);

        // Повторяем вызов с интервалом
        Invoke(nameof(DropEgg), _timeBetweenEggDrops > 0 ? _timeBetweenEggDrops : 2f);
    }

    private void Update()
    {
        // Обновляем позицию дракона
        Vector3 pos = transform.position;
        pos.x += _speed * Time.deltaTime;
        transform.position = pos;

        // Проверяем границы
        if (pos.x < -leftRightDistance)
        {
            _speed = Mathf.Abs(_speed); // Двигаемся вправо
        }
        else if (pos.x > leftRightDistance)
        {
            _speed = -Mathf.Abs(_speed); // Двигаемся влево
        }
    }

    private void FixedUpdate()
    {
        // Случайная смена направления
        if (Random.value < chanceDirection)
        {
            _speed *= -1;
        }
    }
}
```

Тоже самое, но для щита: 

```C#
using System.Collections.Generic;
using UnityEngine;
using UnityEngine.SceneManagement;

public class DragonPicker : MonoBehaviour
{
    [SerializeField] private LoadSettings _settings; // Ссылка на настройки
    [SerializeField] private GameObject energyShieldPrefab; // Префаб энергетического щита
    [SerializeField] private float energyShieldBottomY = -6f; // Координата Y для размещения щитов
    [SerializeField] private float energyShieldRadius = 1.5f; // Радиус размещения щитов

    private int _numEnergyShield; // Количество щитов
    private List<GameObject> _shieldList = new(); // Список текущих щитов

    private void OnEnable()
    {
        _settings.CoroutineIsStopped += UpdateShields; // Подписка на событие обновления
    }

    private void OnDisable()
    {
        _settings.CoroutineIsStopped -= UpdateShields; // Отписка от события при отключении
    }

    private void CreateShields()
    {
        // Очистка текущего списка щитов
        foreach (var shield in _shieldList)
        {
            if (shield != null) Destroy(shield);
        }
        _shieldList.Clear();

        // Создание нового набора щитов
        for (int i = 0; i < _numEnergyShield; i++)
        {
            GameObject shieldGO = Instantiate(energyShieldPrefab);
            float angle = i * Mathf.PI * 2f / _numEnergyShield; // Расположение по кругу
            Vector3 pos = new Vector3(
                Mathf.Cos(angle) * energyShieldRadius,
                energyShieldBottomY,
                Mathf.Sin(angle) * energyShieldRadius
            );
            shieldGO.transform.position = pos;
            _shieldList.Add(shieldGO);
        }
    }

    private void UpdateShields()
    {
        _numEnergyShield = _settings.CountShield; // Получение количества щитов из настроек
        CreateShields(); // Создание щитов
    }

    public void DragonEggDestroyed()
    {
        // Уничтожение всех яиц
        foreach (var egg in GameObject.FindGameObjectsWithTag("Dragon Egg"))
        {
            Destroy(egg);
        }

        // Уничтожение последнего щита
        if (_shieldList.Count > 0)
        {
            int lastIndex = _shieldList.Count - 1;
            Destroy(_shieldList[lastIndex]);
            _shieldList.RemoveAt(lastIndex);
        }

        // Проверка на конец игры
        if (_shieldList.Count == 0)
        {
            SceneManager.LoadScene("_0Scene"); // Загрузка сцены при проигрыше
        }
    }
}
```
Также измененный код для заполнения таблицы
``` Python3
import gspread
import matplotlib.pyplot as plt
import numpy as np

# Авторизация через сервисный аккаунт
gc = gspread.service_account(filename="dragonpicker-8110f907cb46.json")
sh = gc.open("DragonPicker DA")

def update_indexs():
    """Обновление индексов в столбце A."""
    for i in range(1, 11):
        sh.sheet1.update_acell("A" + str(i+1), str(i))

def update_dragon_speed(percent: str):
    """Обновление скорости дракона на основе процента изменения."""
    current_speed = float(sh.sheet1.acell("B2").value)
    x = [current_speed]
    y = [1]
    for i in range(3, 12):
        current_speed *= 1 + (float(percent) / 100)  # Избегаем строковых преобразований
        current_speed = round(current_speed, 2)  # Округление до 2 знаков
        sh.sheet1.update_acell("B" + str(i), str(current_speed))
        x.append(current_speed)
        y.append(i)
    
    # Построение графика
    plt.plot(x, y, label=f"Dragon Speed ({percent}%)")
    plt.xlabel("Index")
    plt.ylabel("Speed")
    plt.title("Dragon Speed Update")
    plt.legend()

def update_time_egg_drop(percent: str):
    """Обновление времени падения яйца на основе процента изменения."""
    tail = 100 - int(percent)
    current_time = float(sh.sheet1.acell("C2").value)
    x = [current_time]
    y = [1]
    for i in range(3, 12):
        current_time *= 1 - (tail / 100)  # Уменьшаем время на основе процента
        current_time = round(current_time, 2)  # Округление до 2 знаков
        sh.sheet1.update_acell("C" + str(i), str(current_time))
        x.append(current_time)
        y.append(i)
    
    # Построение графика
    plt.plot(x, y, label=f"Time Egg Drop ({percent}%)")
    plt.xlabel("Index")
    plt.ylabel("Time")
    plt.title("Time Egg Drop Update")
    plt.legend()

def update_shields_count():
    """Обновление количества щитов."""
    current_count = int(sh.sheet1.acell("D2").value)
    repeat = 1
    x = [current_count]
    y = [1]
    for i in range(3, 12):
        sh.sheet1.update_acell("D" + str(i), str(current_count))
        repeat += 1
        if repeat == 3:
            current_count -= 1
            repeat = 0
        x.append(current_count)
        y.append(i)
    
    # Построение графика
    plt.plot(x, y, label="Shields Count")
    plt.xlabel("Index")
    plt.ylabel("Shield Count")
    plt.title("Shields Count Update")
    plt.legend()

if __name__ == "__main__":
    update_dragon_speed("20")
    update_time_egg_drop("10")
    update_shields_count()

    # Показываем все графики
    plt.tight_layout()
    plt.show()
```
## Выводы
В результате проделанной работы я вроде научился работать с параметрами баланса, визуализировать их, расчитывать а также использовать алгоритм изменения параметров Unity с помощью Python и Google Sheets.
  
