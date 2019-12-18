# Introduction to writing Tasks

В этом уроке мы объясним основные элементы Bamboo Task API,
включая запись в лог и установку applicable result state.

Вначале создайте новый плагин.
Загрузите копию последней версии Plugin SDK
и установите ее на свой компьютер.
Запустите команду atlas-create-bamboo-plugin из командной строки.

Когда вас попросят указать groupId, artifactId, package
просто введите myfirstplugin
и оставьте значения всех остальных параметров 
в виде значений предлагаемых по умолчанию.


# Написашие вашей первой задачи

Создайте новый класс с именем MyFirstTask
в пакете myfirstplugin (myfirstplugin/src/main/java/myfirstplugin)
и реализуйте интерфейс TaskType для класса MyFirstTask.
Измените свой класс, чтобы он выглядел так, как показано ниже.

```java
package myfirstplugin;

import com.atlassian.bamboo.build.logger.BuildLogger;
import com.atlassian.bamboo.task.TaskContext;
import com.atlassian.bamboo.task.TaskException;
import com.atlassian.bamboo.task.TaskResult;
import com.atlassian.bamboo.task.TaskResultBuilder;
import com.atlassian.bamboo.task.TaskType;

public class MyFirstTask implements TaskType
{
    @Override
    public TaskResult execute(final TaskContext taskContext) throws TaskException
    {
        final BuildLogger buildLogger = taskContext.getBuildLogger();

        buildLogger.addBuildLogEntry("Hello, World!");

        return TaskResultBuilder.newBuilder(taskContext).success().build();
    }
}

```

# Что мы видим здесь?

## TaskType
TaskType является минимальным необходимым классом для создания задачи.
Именно здесь происходит тяжелая работа всех ваших плагинов,
и все задачи дожны быть способны запускаться на удаленном агенте.

В нашем примере мы получаем предоставленный BuildLogger
из TaskContext и пишем в лог строку "Hello world".
Конечно, задачи, котоыре вы моежте разработать, могут быть
намного более сложными чем эта.

## TaskContext

TaskContext предоставляет контектстную информацию о том,
где и как выполянется задача.
Из этого объекта можно получить ссылку на конфигурацию,
build log, рабочий каталог
и информацию о выполнии job.


## TaskResult and TaskResultBuilder

Чтобы сообщить Bamboo, успешно ли выполнена ваша задач
или нет, вам нужно вернуть экземпляр TaskResult.
Для облегчения создания TaskResult мы предоставляем TaskResultBuilder.
TaskResultBuilder использует шаблон Builder для создания и поддержки TaskResult.

Нпример, если условие local variable condition истинно,
то TaskResult, вычисляемый при вызове build(), будет успешным,
даже если мы изначально установили его как Failed.

```
public TaskResult execute(final TaskContext taskContext) throws TaskException
{
    final TaskResultBuilder builder = TaskResultBuilder.create(taskContext).failed(); //Initially set to Failed.

    ...

    if (condition)
    {
        builder.success();
    }

    final TaskResult result = builder.build();

    return result;
}

```

# Регистрация вашей задачи в системе плагинов

В файле atlassian-plugin.xml 
просто добавьт следующий элемент:

```
<taskType key="myFirstTask" name="My First Task" class="myfirstplugin.MyFirstTask">
  <description>A task that prints 'Hello, World!'</description>
</taskType>
```

Это позволит системе плагинов знать,
как называется задача,
знать ее описание,
ключ и класс отвечающий за реализацию задачи.
Все это необходимо для того чтобы задачу можно было использовать в Bamboo.
Для получения дополнительной информации возможных значениях дескриптора задачи
смотрите [The Task Type Module Definition](http://confluence.atlassian.com/display/BAMBOO/Tasks+Overview?_ga=2.243792647.385607501.1576507130-1549702659.1570801910#TasksOverview-TheTaskTypeModuleDefinition)


# Запуск плагина MyFirstTask
В корневой директории нашего проекта, выполните в командной строке atlas-run.
Эта команда скомплирует ваш плагин,
запустит его тесты
и запустит Bamboo с установленным плагином.
Вы можете увидеть, что Bamboo закончил загрузку плагина,
когда увидите в логе следующие сообщения:

```
[WARNING] [talledLocalContainer] May 17, 2011 1:40:54 PM org.apache.catalina.startup.Catalina start
[WARNING] [talledLocalContainer] INFO: Server startup in 40924 ms
[INFO] [talledLocalContainer] Tomcat 6.x started on port [6990]
[INFO] bamboo started successfully and available at http://localhost:6990/bamboo
[INFO] Type CTRL-C to exit
```

# Проверим что плагин загрузился корректно

Перейдите по адресу http://localhost:6990/bamboo в браузере,
затем перейдите в Administration - App management, в верхнем правом углу страницы Bamboo
и вы должны увидеть страницу со списком плагинов,
и увидеть разрабатываемый плагин в списке плагинов установленных пользователем (user installed plugins)

ИЗОБРАЖЕНИЕ

# Запустим нашу задачу

Создайте новый план. Выберите задачу "My first task" из списка задач.
Добавьте ее в Default Job
и запустите план на выполнение.

После выполнения, вы можете просмотреть логи
и в них должны увидеть вывод нашей задачей строки "Hello, world!"

ИЗОБРАЖЕНИЕ
