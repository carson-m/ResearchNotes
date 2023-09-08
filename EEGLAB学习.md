# EEGLAB学习

## 重要概念的介绍

见https://eeglab.org/tutorials/ConceptsGuide/

## EEG预处理工作流程

<img src="C:\Users\Carson Ma\AppData\Roaming\Typora\typora-user-images\image-20230709202948715.png" alt="image-20230709202948715" style="zoom:35%;" />

## 事件的导入

EEG以矩阵形式存储，共n行。其中n-1行每行存储一个channel的脑电信号，余下一行为事件行，用脉冲信号的幅度记录当前事件的状态（例：1 (stimulus onset), 2 (subject response), and 0 (other)）。导入时依据事件行的信息区分刺激与反应，EEGLAB导入事件时不会把事件行一起导入。导入后，Select menu item Edit → Event values to inspect the imported event types and latencies.

也可从matlab array或txt文件导入

## 事件的几个要素

- **type** - specifies the type of event. For example, ‘square’ in the example above is a stimulus type, ‘rt’ is a subject button-press (i.e., reaction-time) event, etc… In continuous datasets, EEGLAB may add events of type ‘boundary’ to specify data boundaries (breaks in the continuous data).

- **latency** - contains event latencies. The latency information is displayed in seconds for continuous data or in milliseconds relative to the epoch’s time-locking event for epoched data. As we will see in the event scripting section, the latency information is stored internally in data samples. These may be fractional samples in case the time resolution of events exceeds the data resolution. Events must always be in order of increasing latencies.

- **duration** - duration of the event. This information is displayed in seconds for continuous data, and in milliseconds for epoched data. Internally, duration is stored in data samples.

- **urevent** - contains indices of events in the original (‘ur’ in German) event structure. The first time events are imported, they are copied into a separate structure called the ‘urevent’ structure. This field is hidden in the graphic interface (above) since it should not be casually modified. This field is described in detail in the event scripting tutorial.

- **epoch** - indices of the data epochs (if any) the event falls within. This field is only present for data for which data epochs have been extracted (which is not the case here since the data is continuous).

## 重参考

见https://eeglab.org/tutorials/ConceptsGuide/rereferencing_background.html

## Baseline Removal

Removing a mean baseline value from each epoch is **useful when baseline differences between data epochs (e.g., those arising from low-frequency drifts or artifacts) are present**. These are not meaningfully interpretable, but if left in the data, could skew the data analysis. However, **high-pass filtering data is also a form of baseline correction, rendering the current step optional**. It is also essential to remember that **baseline correction introduces random offsets in each channel, something ICA and source reconstruction algorithms cannot easily cope with**. Baseline correction should thus be used with caution and *short baseline windows (on the order of 100 milliseconds) avoided when possible*.
