# Writing a Task with a User Interface

Пример плагина "Hello World"
больше не предоставляется по умолчанию в Atlassian SDK.
Но вы можете создать аналогичный с помощю этого учебника:
[Introduction to writing tasks](https://developer.atlassian.com/server/bamboo/introduction-to-writing-tasks/)

Ранее плагин "Hello World" - пример плагина Bamboo,
который поставлялся как проект по умолчанию в Atlassian Plugin SDK.
В этом уроке мы разберем плагин и рассмотрим как создать пользовательский интерфейс для плагина
и добавить валидацию.

# Создаем новый плагин
Сначала загружаем копию Atlassian SDK
и устанавливаем ее на своем копьютере.

После того как Atlassian SDK установлена,
запустите команду atlas-create-bamboo-plugin
из командной строки.

Когда вас попросят указать groupId, artifactId, packageId
просто введите "helloworld"
и оставьте значения других параметров в виде значений 
предоставляемых по умолчанию.

Что такое плагин Hello World?

Плагин Hello world это пример плагина из Atlassian SDK,
который демонстрирует, как создать плагин Task
с пользовательским интерфейсом, валидацией и интернационализацией.

Плагин очень прост.

Он позволяет пользователю в Bamboo настроить задачу так,
чтобы она печатала введенное в настройках плагина значение в лог
и после этого задача будет возвращать успешный результат (TaskResult).


Попробуйте это.
Вы должны иметь права администратора, чтобы это сделать.

Первая учетная запись, которую вы зарегистрировали на своем экземпляре сервера,
должна иметь такие права по умолчанию.
В противном случае вы можете попробовать использовать логин admin 
c паролем admin.


Чтобы попробовать как работает плагин
запустите команду atlas-run командной строки
в корневой директории проекта
(проект мы сгенерировали ранее).

Когда Bamboo запущен, создайте новый План с типом репозитория "None"
и настройе Job, добавив туда задачу "Hello World"

ИЗОБРАЖЕНИЕ
https://developer.atlassian.com/server/bamboo/images/helloworld.jpg


Попробуйте удалить значение по умолчанию в поле "Say" и сохранить.

Вы должны увидеть ошибку валидации, как это показано ниже:


ИЗОБРАЖЕНИЕ
https://developer.atlassian.com/server/bamboo/images/validation-error.jpg

Это проверка, которую мы определили в Configurator.

Мы покажем вам, как добавить вашу собственную проверку в ближайшее время.

Измените значение поля "Say" на "Hello World!"

затем сохраните задачу и запустите свой план.

Как только план завершит выполнение, вы увидите,

что плагина запускается и печатает "Hello, World"

в default Job логе

ИЗОБРАЖЕНИЕ
https://developer.atlassian.com/server/bamboo/images/helloworldresult.jpg

# Разберем структуру плагина

Теперь мы разберем плагин по компонентно 

и объясним, как они работают вместе.


## TaskType

Мы рассмотрели, что такое TaskType и как он работает,
в руководстве [Introduction to Tasks ](https://developer.atlassian.com/server/bamboo/introduction-to-writing-tasks)

Эта задача извлекает значение из configuration map

по ключу "say" и добавляет новую запись со своим значением в build log

а после этого возвращает успешный результат выполнения (TaksResult)


Пример, ExampleTask.java

```java
public class ExampleTask implements TaskType
{
    @NotNull
    @java.lang.Override
    public TaskResult execute(@NotNull final TaskContext taskContext) throws TaskException
    {
        final BuildLogger buildLogger = taskContext.getBuildLogger();

        final String say = taskContext.getConfigurationMap().get("say");

        buildLogger.addBuildLogEntry(say);

        return TaskResultBuilder.create(taskContext).success().build();
    }
}
```

# Templates

Давайте посмотрим на шаблоны в Hello World plugin.

Bamboo использует фреймворк WebWork и язык шаблонов Freemarker

для визуализации своего пользовательского интерфейса.

В WebWork доступно множество тегов,

которые можно использовать из freemarker

для мапинга значенией из Configurator на элементы в пользовательском интерфейсе 

(чуть позже мы поговорим о конфигураторе (Configurator))


Пример из editExampleTask.ftl

```
[@ww.textfield labelKey="helloworld.say" name="say" required='true'/]
```

Этот шаблон предоставляет configuration form для нашей задачи.

Мы используем textField здесь,

чтобы обеспечить ввод значений.

Значение labelKey helloworld.say
сопоставляется со свойством из english.properties файла

который предоставляет перевод,

а имя указывает, какое значение textField должно быть в Конфигураторе (Configurator)

В этом случае мы всегда хотим,

чтобы пользователь указал значение для поля,

поэтому мы делаем наше поле обязательным (required),

установим для атрибута required значение true.


## TaskConfigurator

А что представляет собой TaskConfigurator?

TaskConfigurator - это классс,

который контролирует, какие объекты и значения

доступны при отображении пользовательского интерфейса,

и как сохраняются и валидируются вводимые данные.

В нашем примере мы используем класс AbstractTaskConfigurator

вместо интерфейса TaskConfigurator,

так как возможно вам не потребуется реализовать все его элементы интерфейса.


## Сохранение введенных настроек

Чтобы сохранить вашу конфигурацию, 

вам нужно переопредить или реализовать метод [generateTaskConfigMap](http://docs.atlassian.com/atlassian-bamboo/latest/com/atlassian/bamboo/task/TaskConfigurator.html?_ga=2.197564333.385607501.1576507130-1549702659.1570801910#generateTaskConfigMap(com.atlassian.bamboo.collections.ActionParametersMap,%20com.atlassian.bamboo.task.TaskDefinition)) 

в вашем TaskConfigurator.

Пример из ExampleTaskConfigurator.java

```java
public Map<String, String> generateTaskConfigMap(@NotNull final ActionParametersMap params, @Nullable final TaskDefinition previousTaskDefinition)
{
    final Map<String, String> config = super.generateTaskConfigMap(params, previousTaskDefinition);

    config.put("say", params.getString("say"));

    return config;
}
```

Когда пользователь сохраняет задачу,

вызывается метод generateTaskConfigMap

и предоставляются ActionParametersMap и TaskDefinition


ActionParametersMap содержит все ключи параметров формы

и значения из полей ввода расположенных в шаблоне editExampleTask.ftl


TaskDefinition - это сохраненное состояние задачи Bamboo.

Он содержит user description и map с ключами/значениями.

Следует отметить, что если создается задача,

экземпляр TaskDefinition здесь имеет значение null,

но если он редактируется, 

то он содержит предыдущее состояние задачи, до того момента как пользователь ее изменил.


Помните поле с именем "say" в editExampleTask.ftl?

Значение, установленное пользователем для этого поля,

доступно в ActionParametersMap.


Чтобы сохранить значение в новом TaskDefinition,

мы должны вернуть новый экземпляр map,

которая содержит это значение.


Для простоты мы используем тот же ключ для этого значения,

который использовался в качестве имени поля.


## Проверка входных данных настроек

Пример из ExampleTaskConfigurator.java

```java

public void validate(@NotNull final ActionParametersMap params, @NotNull final ErrorCollection errorCollection)
{
    super.validate(params, errorCollection);

    final String sayValue = params.getString("say");
    if (StringUtils.isEmpty(sayValue))
    {
        errorCollection.addError("say", textProvider.getText("helloworld.say.error"));
    }
}
```

Перед вызовом generateTaskConfigMap 

происходит валидация ActionParametersMap ,

чтобы убедиться, что в ней данной структуре содержатся правильные значения,

необходимые для создания нового TaskDefinition в generateTaskConfigMap.

Проверка проста.

Если поле не проходит ваши правила проверки Task,

вы просто вызываете addError для ErrorCollection с именем поля,

которое не прошло проверку,

и с сообщение о том, почему проверка не прошла успешно.

## Заполнение Create и Edit

В этом методе вы можеет установить для вашей задачи настройки по умолчанию,

которые будут действовать при создании задачи.

Для этого необходимо заполнить context map ключами и значениями,

которые вы хотите чтобы эти ключи содержали.

Пример из ExampleTaskConfigurator.java

```java
@Override
public void populateContextForCreate(@NotNull final Map<String, Object> context)
{
    super.populateContextForCreate(context);

    context.put("say", "Hello, World!");
}

```

После того как ваша задача была создана
populateContextForEdit может быть использован для отображения

значений настроек на задачу (Task)

Просто заполните данную context map значениями,

хранящимися в map в TaksDefinition (это значения,

которые устанавливает для новой map из ActionParamentersMap в 

методе generateTaskConfigMap)

Как и populateContextForCreate,

ключи context map должны соответствовать именам полей в ваших шаблонах.


Пример из ExampleTaskConfigurator.java

```java
@Override
public void populateContextForEdit(@NotNull final Map<String, Object> context, @NotNull final TaskDefinition taskDefinition)
{
    super.populateContextForEdit(context, taskDefinition);

    context.put("say", taskDefinition.getConfiguration().get("say"));
}
```
## Регистрация конфигуратора (Configurator) и ассоцированных шаблонов в Plugin system

Пример из atlassian-plugin.xml:

```xml
<taskType name="helloworld" class="helloworld.ExampleTask" key="test">
  <description>A simple Hello World Task</description>
  <!-- Categories available in 3.1: "builder", "test" and "deployment"
  <category name=""/>
  -->
  <configuration class="helloworld.ExampleTaskConfigurator"/>
  <resource type="freemarker" name="edit" location="editExampleTask.ftl"/>
</taskType>
```

## Интернационализация и TextProvider

В нашем примере мы использовуем injected TextProvider в ExampleTaskConfiguratior.java

для загрузки строки helloworld.say.error из файла интернационализации english.properties,

когда проверка не удалась.


Чтобы зарегистрировать english.properties в системе плагинов,

добавьте строку в atlassian-plugin.xml 

и сделаете это примерно таким образом:

Пример из atlassian-plugin.xml:

```xml
<resource type="i18n" name="helloworld language" location="english"/>
```
