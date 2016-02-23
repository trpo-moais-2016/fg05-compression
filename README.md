# Задание 5*. Фрактальное сжатие
Написать программу, позволяющую производить фрактальное кодирование и декодирование изображений в градациях серого.
При этом:
* использовать квадратные ранговые блоки одинакового размера, стороны которых параллельны соответствующим сторонам изображения
* использовать квадратные доменные блоки одинакового размера, стороны которых параллельны соответствующим сторонам изображения
* использовать среднеквадратичную метрику и метод наименьших квадратов при сопоставлении доменных и ранговых блоков

## Входные/выходные данные
### Кодирование
* Вход
    - квадратное изображение в градациях серого, размер которого кратен размеру рангового блока
    - размер рангового блока
    - размер доменного блока, больший размера рангового блока
    - смещение между соседними доменными блоками

* Выход
    - закодированное изображение, т.е. массив байт или объект, содержащий информацию о сопоставлении доменных блоков ранговым, параметры построения доменных и ранговых блоков
    
### Декодирование
* Вход
    - закодированное изображение
    - произвольное квадратное стартовое изображение или желаемый размер стороны декодированного изображения

* Выход    
    - раскодированное изображение желаемого размера или с размерами заданного стартового изображения

## Алгоритм кодирования
* По заданному изображению строятся ранговые блоки
* По заданному изображению строятся доменные блоки
* Для каждого рангового блока пытаемся найти наиболее подходящий доменный блок
* Добавляем в массив сопоставлений информацию о найденном доменном блоке и требующемся преобразовании контраста и яркости
* Собираем результат из массива сопоставлений и другой информации в некотором формате.

## Построение рангов
* Ранги строятся как квадраты заданного размера, покрывающие все изображение

![Ранги](/images/rangesSimple.png)

## Построение доменов
* Домены строятся как квадраты заданного размера с заданными горизонтальным и вертикальтным смещением друг относительно друга
* Для упрощения задачи будем считать вертикальное смещение `dH` равным горизонтальному смещению `dW`
* При этом соседние домены могут перекрываться

![Домены перекрываются](/images/domainsSimple1.png)

* А могут не перекрываться

![Домены не перекрываются](/images/domainsSimple2.png)

* Выходящие за границы изображения домены игнорируются

## Алгоритм сопоставления доменных и ранговых блоков
Имеется ранговый блок. Необходимо найти подходящий доменный блок из имеющихся.
Для этого производим сопоставление с этим ранговым блоком следующим образом:
* Сжимаем доменный блок до размеров рангового блока
* Вычисляем методом наименьших квадратов оптимальные параметров контрастности и яркости
* Вычисляем квадрат расстояния между ранговым и доменным блоками как изображениями
* Если результат лучше уже найденного, то сохраняем, иначе продолжаем сопоставление

## Среднеквадратичная метрика и метод наименьших квадратов
Для определения разницы изображений предлагается использовать среднеквадратическую метрику:

![Среднеквадратичная метрика](/images/rms.png)

В этом случае при сравнении рангового блока с доменным, размеры которого предварительно были уменьшены до размеров рангового, следует минимизировать следующий функционал:

![Расчет качества](/images/quality.png)

За счет подбора коэффициента `s`, имеющего смысл контраста, и коэффициента `o`, имеющего смысл яркости.

Быстро найти оптимальные значения контраста и яркости можно методом наименьших квадратов:

![Яркость и контраст](/images/formula1.png)

![Коэффициенты](/images/formula2.png)

## Результат
После окончания кодирования должен получится некий массив байт известной структуры, который можно сохранить в файл.
Это и есть закодированное изображение.

В нем необходимо сохранить информацию
* как построить ранговые блоки для произвольного стартового изображения
* как построить доменные блоки для произвольного стартового изображения
* какие доменные блоки соответсвуют ранговым с какими значениями яркости и контрастности

## Алгоритм декодирования
* Строятся доменные блоки по размерам исходного изображения в соответствии с сохраненными параметрами
* Строятся ранговые блоки по размерам исходного изображения в соответствии с сохраненными параметрами
* Доменные и ранговые блоки растягиваются пропорционально отношению желаемых размеров изображения к исходным размерам, если эти размеры не равны
* Информация о соответствующих ранговых блоках, доменных блоках, а также преобразованиях контраста и яркости объединяется в массив правил по одному на каждый ранговый блок
* К стартовому изображению применяются получившиеся правила, в результате чего строится новое изображение. Это первая итерация декодирования
* Последующие итерации декодирования заключаются в применении правил к изображению, получившимуся в предыдущей итерации, в результате чего строится новое изображение
* Итерации декодирования продолжаются до тех пор, пока не достигнуто желаемое качество изображения, или изображение перестает значительно меняться, или превышено ограничение на общее их количество

## Применение правил декодирования
Применение правил заключается в том, что для каждого правила берется область нового изображения, соответствующая ранговому блоку из правила, и заполняется значениями цветов старого изображения из доменного блока, которые предварительно изменяются на заданную в правиле контрастность и яркость.
Так как ранговые блоки полностью покрывают новое изображение, то оно полностью отрисовывается.