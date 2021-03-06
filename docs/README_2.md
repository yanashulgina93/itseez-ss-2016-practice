# Практика 2. Основы работы с библиотекой OpenCV. Модули core, imgproc и highgui

[![Feedback](feedback.png)][feedback_day2]

## Цели

__Цель данной работы__ - изучить базовые примитивы модуля 
opencv_core и простейшие операции обработки изображений, входящие в состав
модуля opencv_imgproc, научиться разрабатывать приложения с графическим 
интерфейсом средствами модуля opencv_highgui.

## Задачи

__Основные задачи__

  1. Реализовать метод конвертации заданной прямоугольной области
     цветного изображения в оттенки серого.
  1. Реализовать метод размытия заданной прямоугольной области
     цветного изображения с помощью медианного фильтра.
  1. Реализовать метод определения ребер в заданной области
     изображения с использованием метода Канни.
  1. Реализовать метод пикселизации заданной области изображения.
  1. Разработать приложение, которое на вход принимает изображение
     и метод преобразования заданной области изображения (см.
     предыдущие пункты). Приложение должно обеспечивать следующий
     функционал:
     
     1. Отрисовка исходного изображения.
     1. Выделение области для преобразования на исходном изображении
        с использованием курсора мыши.
     1. Отрисовка результата выполнения преобразования.

__Дополнительные задачи__

  1. Разработать приложение, обеспечивающее аналогичный функционал
     преобразования области изображения для случая, когда на входе
     имеется видео.
  1. Разработать приложение, обеспечивающее аналогичный функционал
     для случая, когда на входе имеется видео, поступающее с видеокамеры.
  1. Разработать приложение, которое на вход принимает изображение
     и обеспечивает следующий функционал:
     
     1. Отрисовка исходного изображения.
     1. Установка точки на исходном изображении с использованием
        курсора мыши.
     1. Построение и отображение выпуклой оболочки набора точек,
        поставленных на изображении. Замечание: при добавлении очередной
        точки выпуклая оболочка может пересчитываться и перерисовываться
        в режиме online.
     
  Пример работы приложения, которое решает данную задачу, можно посмотреть
  на [YouTube](https://youtu.be/atsnS1SNiFM).
  
## Общая структура программного модуля

В лабораторной работе основные задачи решаются на базе программного
модуля `image_processing` библиотеки `summer_school_2016_lib`. Модуль
включает заголовочный файл `include\image_processing.hpp`
и исходный файл`src\image_processing.cpp`. Модуль содержит объявление
абстрактного класса `ImageProcessor`.

```cpp
class ImageProcessor {
 public:
   virtual cv::Mat CvtColor(const cv::Mat &src, const cv::Rect &roi) = 0;
   virtual cv::Mat Filter(const cv::Mat &src, const cv::Rect &roi, 
                          const int size) = 0;
   virtual cv::Mat DetectEdges(const cv::Mat &src, const cv::Rect &roi, 
                               const int filter_size, const int low_threshold,
                               const int ratio, const int kernel_size) = 0;
   virtual cv::Mat Pixelize(const cv::Mat &src, const cv::Rect &roi, 
                            const int divs) = 0;
};
```

Класс имеет следующие чисто виртуальные методы:

  1. `CvtColor` - метод конвертации прямоугольной области `roi` исходного
     изображения `src` в оттенки серого.
  1. `Filter` - метод медианной фильтрации прямоугольной области `roi` 
     исходного изображения `src` с помощью фильтра с ядром 
     размера `size x size`.
  1. `DetectEdges` - метод для выделения ребер в прямоугольной области `roi`
     исходного изображения `src` с использованием детектора ребер Канни.
     Входные параметры: `filter_size` - размер ядра линейного фильтра, 
     `low_threshold` и `low_threshold * ratio` пороги метода Канни,
     `kernel_size` - размер ядра фильтра Собеля.
  1. `Pixelize` - метод пикселизации прямоугольной области `roi`
     исходного изображения `src` (телевизионный эффект, используемый
     для того, чтобы скрыть лица людей) с количеством делений
     `divs` по каждой оси.
  
## Общая последовательность действий

  1. Разработать объявление наследника `ImageProcessorImpl`
     класса `ImageProcessor`.
  1. Последовательно реализовать методы класса `ImageProcessorImpl`.
  1. Скопировать `samples\template_demo.cpp` в директорию `samples`.
  1. Переименовать копию примера в `samples\imgproc_demo.cpp`.
  1. Разработать приложение `samples\imgproc_demo.cpp` в соответствии
     с перечнем требований, представленным в описании основных задач.
  1. Разработать приложения, решающие дополнительные задачи.

## Детальная инструкция по выполнению работы

  1. Разработать объявление наследника `ImageProcessorImpl` класса
     `ImageProcessor` и поместить его в файл `include\image_processing.hpp`.

  ```cpp
  class ImageProcessorImpl : public ImageProcessor {
  public:
    virtual cv::Mat CvtColor(const cv::Mat &src, const cv::Rect &roi);
    virtual cv::Mat Filter(const cv::Mat &src, const cv::Rect &roi,
                           const int size);
    virtual cv::Mat DetectEdges(const cv::Mat &src, const cv::Rect &roi,
                                const int filter_size, const int low_threshold,
                                const int ratio, const int kernel_size);
    virtual cv::Mat Pixelize(const cv::Mat &src, const cv::Rect &roi,
                             const int divs);
  };
  ```

  1. Реализовать метод `ImageProcessorImpl::CvtColor`. Метод предполагает
     выполнение следующей последовательности действий:
  
     1. Создать копию `src_copy` исходного изображения `src`.
     1. Выделить подматрицу `src_copy_roi` из копии `src_copy`, соответствующую
        области `roi`.
     1. Сконвертировать подматрицу `src_copy_roi` в оттенки серого, результат
        записать в матрицу `dst_gray_roi`. Примечание: необходимо использовать
        функцию `cvtColor`.
     1. Создать вектор матриц `vector<Mat> channels`, соответствующих
        каналам результирующей области преобразованного изображения `dst_roi`.
     1. Сформировать `dst_roi` посредством склеивания трех каналов, каждый канал
        соответствует `dst_gray_roi`. Примечание: необходимо использовать
        функцию `merge`.
     1. Скопировать `dst_roi` в подматрицу `src_copy_roi`.
     1. Вернуть `src_copy`.
  
  1. Реализовать метод `ImageProcessorImpl::Filter`. Метод предполагает
     выполнение следующей последовательности действий:
     
     1. Создать копию `src_copy` исходного изображения `src`.
     1. Выделить подматрицу `src_copy_roi` из копии `src_copy`, соответствующую
        области `roi`.
     1. Вызвать функцию медианной фильтрации `medianBlur` для выделенной
        области `src_copy_roi`.
     1. Вернуть `src_copy`.
     
  1. Реализовать метод `ImageProcessorImpl::DetectEdges`.
  
     1. Выделить подматрицу `src_roi` из копии `src`.
     1. Сконвертировать матрицу `src_roi` в оттенки серого, результат записать
        в матрицу `src_gray_roi`.
     1. Отфильтровать `src_gray_roi` с использованием линейного фильтра, результат
        записать в матрицу `gray_blurred`. Примечание: для фильтрации можно
        использовать функцию `blur`.
     1. Построить ребра `detected_edges` на изображении `gray_blurred` с помощью
        функции `Canny`.
     1. Создать матрицу `dst`.
     1. Скопировать изображение `src` в `dst`.
     1. Выделить подматрицу `dst_roi` из `dst` в соответствии с областью `roi`.
     1. Обнулить все значения в подматрице `dst_roi`. Примечание: необходимо
        использовать статический метод `all` класса `Scalar`.
     1. Скопировать `src_roi` в `dst_roi` в соответствии с маской `detected_edges`.
     1. Вернуть `dst`.
  
  1. Реализовать метод `ImageProcessorImpl::Pixelize`.
  
     1. Создать копию `src_copy` исходного изображения `src`.
     1. Выделить подматрицу `src_сopy_roi` из копии `src_copy`.
     1. Определить размеры блока пикселизации `block_size_x = roi.width / divs`,
        `block_size_y = roi.height / divs`.
     1. Реализовать обход пикселей выделенной области `roi` по всем `x` и `y`
        с шагами `block_size_x` и `block_size_y` по осям Ox и Oy соответственно.
          
          1. Для каждого положения с координатами (`x`, `y`) в выделенной области
             взять вложенную область `src_roi_block` размера 
             (`block_size_x`, `block_size_y`).
          1. Выполнить размытие области `src_roi_block` с помощью линейного 
             фильтра с ядром размера (`block_size_x`, `block_size_y`).
          
     1. Вернуть `src_copy`.
  
  1. Скопировать `samples\template_demo.cpp` в директорию `samples`.
  1. Переименовать копию примера в `samples\imgproc_demo.cpp`.
  1. Разработать приложение `samples\imgproc_demo.cpp` в соответствии
     с требованиями, перечисленными в основных задачах.
     
     1. Создать массив опций приложения: `image` - название исходного
        изображения; `gray` - флаг, показывающий, что необходимо выполнить
        преобразование области в оттенки серого; `median` - флаг,
        показывающий, что необходимо выполнить медианную фильтрацию области;
        `edges` - флаг, показывающий, что необходимо построить ребра
        для выделенной области; `pix` - флаг, показывающий, что необходимо
        выполнить пикселизацию.

     ```cpp
     const char* kOptions =
        "{ @image         | <none> | image to process            }"
        "{ gray           |        | convert ROI to gray scale   }"
        "{ median         |        | apply median filter for ROI }"
        "{ edges          |        | detect edges in ROI         }"
        "{ pix            |        | pixelize ROI                }"
        "{ h ? help usage |        | print help message          }";
     ```

     1. Создать структуру для хранения состояния мыши `MouseCallbackState`:
        `point_first` - левый верхний угол выделенной прямоугольной области; 
        `point_second` - правый нижний угол выделенной прямоугольной области;
        `is_selection_started` - флаг, определяющий, что пользователь зажал
        левую кнопку мыши; `is_selection_finished` - флаг, определяющий, 
        что пользователь отпустил левую кнопку мыши. 

     ```cpp
     struct MouseCallbackState {
        bool is_selection_started;
        bool is_selection_finished;
        Point point_first;
        Point point_second;
     };
     ```

     1. Реализовать обработчик `OnMouse` события нажатия на кнопку мыши.
        
        1. Если зажата левая кнопка мыши (событие `cv::EVENT_LBUTTONDOWN`),
           то необходимо выставить правильные значения флагов 
           `is_selection_started = true`, `is_selection_finished = false`
           и сохранить координаты курсора в `point_first`.
        1. Если левая кнопка мыши освобождена (событие `cv::EVENT_LBUTTONUP`),
           то необходимо выставить правильные значения флагов 
           `is_selection_started = false`, `is_selection_finished = true`
           и сохранить координаты курсора в `point_second`.
        1. Если курсор мыши изменил положения (событие `cv::EVENT_MOUSEMOVE`)
           и при этом `is_selection_finished = false`, то необходимо изменить
           положение точки `point_second`.
        
     1. Реализовать основную функцию. Логика выполнения основной функции следующая:
     
        1. Разобрать аргументы командной строки. Примечание: необходимо
           использовать класс `CommandLineParser` библиотеки OpenCV.
        1. Создать окно для отображения исходного изображения и назначить
           на это окно событие `OnMouse`. Для визуализации выделения области
           можно отрисовать построенный прямоугольник. Примечание: использовать функции
           `namedWindow`, `resizeWindow`, `setMouseCallback`, `imshow` и `waitKey`,
           `rectangle`.
        1. Реализовать цикл ожидания выбора области интереса для обработки.
        1. В зависимости от того, какая опция передана на вход программе,
           вызвать фильтр, соответствующей указанной области.
        1. Создать окно для отображения преобразованного изображения.
        1. Показать результат выполнения преобразования.
        
  1. Разработать приложения, решающие дополнительные задачи.

<!-- LINKS -->
[feedback_day2]: https://docs.google.com/forms/d/1geYzNWX1vb4WVRz5odTI2uJ5bNvNfwsBmCTCqsBGjV8/viewform