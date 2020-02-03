# clothes_detection
Clothes detection model
Перед запуском модели необходимо произвести [установку](https://github.com/tensorflow/models/blob/master/research/object_detection/g3doc/installation.md) необходимых модулей.
## Процесс создания модели происходил в несколько этапов:
1. Сбор и разметка данных
2. Создание TFRecords для обучения
3. Выбор модели и ее конфигурации
4. Обучение и сохранение модели
5. Тестирование результата


## 1. Сбор и разметка данных

В качестве классов для распознавания были выбраны 12 типов одежды (dress, shirt, skirt, jacket, coat, Tshirt, shorts, boots, trousers, sunglasses, shoes, sportshoes). Под данные метки было собрано 1500 соответствующих картинок из которых 90% будет использовано в обучении, а 10% на валидации. Источником послужили сайты по продаже одежды.  
  ![example1](https://github.com/aplusk23/clothes_detection/blob/master/images/image1.jpg)
  ![example2](https://github.com/aplusk23/clothes_detection/blob/master/images/image4.jpg)

Собранные данные далее были размечены bounding box'ами с использованием [LabelImg](https://github.com/tzutalin/labelImg). В результате с каждым файлом с картинкой находился xml файл с разметкой.

## 2. Создание TFRecords для обучения

Для дальнейших действий клонируем репозиторий и осуществляем 
[установку](https://github.com/tensorflow/models/blob/master/research/object_detection/g3doc/installation.md)  
```!git clone https://github.com/tensorflow/models```   
Переходим в директорию ```models/research/object_detection``` и добавляем сюда наш датасет, создав папку images.  
Поскольку Tensorflow использует TFRecords для подачи данных на вход модели, добавляем  [xml_to_csv.py](https://github.com/aplusk23/clothes_detection/blob/master/xml_to_csv.py) и [generate_tfrecord.py](https://github.com/aplusk23/clothes_detection/blob/master/generate_tfrecord.py) в эту же директорию и используем их. Скрипты взяты из репозитория [Raccoon Detector Dataset](https://github.com/datitran/raccoon_dataset) и модифицированы под нашу задачу. В результате получаем train.record и test.record файлы.

## 3. Выбор модели и ее конфигурации

Для обучения используем transfer learning предтренированной модели ssd_mobilenet_v2_coco, которая была выбрана как оптимальная по соотношению скорость - качество (сравнение производится [здесь](https://github.com/tensorflow/models/blob/master/research/object_detection/g3doc/detection_model_zoo.md)).
Скачиваем и распаковываем модель в директорию ```models/research/object_detection```. Затем создаем папку training, в которую поместим labelmap.pbtxt(файл с классами) и config модели (образец находим  в ```models/research/object_detection/samples/configs/``` и редактируем под задачу).

## 4 Обучение и сохранение модели

Запустим обучение при помощи скрипта train. с необходимыми параметрами.  
```!python train.py --logtostderr --train_dir=training/ --pipeline_config_path=training/ssd_mobilenet_v2_coco.config```
Для достижения результата потребуется не менее 25000 шагов.
После этого сохраним полученную модель  
```!python export_inference_graph.py 
           --input_type image_tensor \
           --pipeline_config_path training/ssd_mobilenet_v2_coco.config \
           --trained_checkpoint_prefix training/model.ckpt-25000 \
           --output_directory clothes_detection  
```         
 
## 5. Тестирование результата           

Добавим тестовые [изображения](https://github.com/aplusk23/clothes_detection/tree/master/test_images) в папку ```models/research/object_detection/test_images```
Запустим [object_detection.ipynb](https://github.com/aplusk23/clothes_detection/blob/master/object_detection.ipynb) из директории ```models/research/object_detection```. В результате получим изображения с выделенными классами.  
![example3](https://github.com/aplusk23/clothes_detection/blob/master/example/image3.jpg)
![example4](https://github.com/aplusk23/clothes_detection/blob/master/example/image4.jpg)
![example5](https://github.com/aplusk23/clothes_detection/blob/master/example/image5.jpg)
