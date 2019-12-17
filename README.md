# bamboo-task-plugin
В этом обзоре Task API,
мы разбираем как задачи запускаются,
конфигурируются, 
и как использовать дополнительные необязательные API-интерфейсы,
такие как Executables, Test Parsing, Logging, Capabilities, Requirements
и особенно реализации запуска задач на удаленных агентах (Remote Agents).

Введение
В версии 3.1 мы добавили задачи (Tasks)
Каждый Job теперь может иметь несколько задач.

Таски (задачи) заменили Builder.
Задачи выполняются по порядку на одном агенте.
Если задача не выполняется,
следующие задачи не будут выполнены (Final Tasks являются исключением).
Final tasks всегда будут выполняться,
независимот от состояния предыдущих задач (даже если вы остановите сборку вручную),
поэтому такие задачи можно использовать для операций уборки (clean up)

Задачи выполняются после Pre Build Plugins
и до Post Build Plugins.
То, что вы решите сделать в задаче, зависит только от вас.


Особые соображения по разработке задач.
Люди могут добавить несколько ваших TaskType в один job.
Люди могут поместить вашу задачу в раздел final tasks,
поэтому ваша задача должна предусматривать возможность ситуации,
когда результаты предыдущих задач могут не существовать.

Это должен быть 1 рода plugin, потому что он должен работать на удаленных агентах.


# Определение модуля TaskType

Задачи разработаны так, чтобы их можно было легко расширять.
Чтобы получить базовую задачу и запустить ее,
вам просто нужно реализовать один интерфейс на Java.
Тем не менее, существуют дополнительные точки для расширения
функциональности вашей задачи,
которые позволяют вам добавить экран конфигурации
и другии мощные функциию.

# Минимальный TaskType плагин.

```
<taskType key="task.awesome" name="Awesome Task" class="com.atlassian.bamboo.plugins.MyAwesomeTaskType" />

```

# Расширенная форма TaskType плагина

```
<taskType key="task.builder.maven" name="Maven 1.x" class="com.atlassian.bamboo.plugins.maven.task.Maven1BuildTask">
        <description>Execute one or more Maven 1 goals as part of your build</description>
        <category name="builder"/>
        <category name="test"/>
        <executable key="maven" nameKey="builder.maven.executableName" pathHelpKey="builder.maven.helpPath"/>
        <capabilityDefaultsHelper class="com.atlassian.bamboo.plugins.maven.task.Maven1CapabilityDefaultsHelper"/>
        <configuration class="com.atlassian.bamboo.plugins.maven.task.configuration.Maven1BuildTaskConfigurator" />
        <resource type="freemarker" name="edit" location="com/atlassian/bamboo/plugins/maven/task/configuration/maven1BuildTaskEdit.ftl"/>
        <resource type="freemarker" name="view" location="com/atlassian/bamboo/plugins/maven/task/configuration/maven1BuildTaskView.ftl"/>
</taskType>

```

# Выбор типа TaskType

При настройке джоба, пользователю предлагается средство для выбора типа задачи.
Ваш модуль TaskType должен появиться в этом списке.


ИЗОБРАЖЕНИЕ
https://developer.atlassian.com/server/bamboo/images/taskpicker.jpg

Имя и описание модуля TaskType будут видны,
чтобы помочь пользователю выбрать нужны тип TaskType.
Описание не является обязательным,
однако мы настоятельно рекомендуем включить его.


ИЗОБРАЖЕНИЕ
https://developer.atlassian.com/server/bamboo/images/name-and-description-elements.jpg


Вы также можете указать дополнительные категории для групприровки TaskType в окне выбора,
которое откроется перед пользователем.

Доступные категории: builder, test, deployment.


ИЗОБРАЖЕНИЕ
https://developer.atlassian.com/server/bamboo/images/category-elements.jpg

Когда пользователь выбирает TaskType,
у него будет возможность настроить его (при необходимости),
а затем он будет сохранен как [TaskDefinition](http://docs.atlassian.com/atlassian-bamboo/latest/com/atlassian/bamboo/task/TaskDefinition.html?_ga=2.32896828.385607501.1576507130-1549702659.1570801910).
TaskDefinitions хранятся в [BuildDefinitions](http://docs.atlassian.com/atlassian-bamboo/latest/com/atlassian/bamboo/build/BuildDefinition.html?_ga=2.32896828.385607501.1576507130-1549702659.1570801910) вашей джобы.


# Модель данных
Объекты TaskDefinition - это то, что мы храним в BuildDefinition
и они будут использоваться всякий раз,
когда вы настраиваете вашу задачу.
Когда вы запускаете TaskType,
вам предоставляется много другой полезной информации,
которую вы можете получить из TaskContext.
И TaskDefinition и TaskContext реализуют интерфейс TaskIdentifier,
который можно исопльзовать,
когда требуется только основная информация (core information).

ИЗОБРАЖЕНИЕ
https://developer.atlassian.com/server/bamboo/images/1867811.png

The Task
Минимальное требование для модуля TaskType - это класс реализующий интерфейс TaskType.
TaskTypes должны поддерживать: [BAMBOO:Runnign on Remote Agents.](https://developer.atlassian.com/server/bamboo/tasks-overview/#bamboo-running-on-remote-agents)


ИЗОБРАЖЕНИЕ
https://developer.atlassian.com/server/bamboo/images/tasktypeclass.jpg

Интерфейс TaskType имеет только 1 метод.


```
/**
     * Execute the task
     *
     * @param taskContext representing any context and configuration of the Task
     * @return a {@link TaskResult} representing the status of the task execution
     * @throws TaskException
     */
    @NotNull
    TaskResult execute(@NotNull final TaskContext taskContext) throws TaskException;

```

TaskContext

В котором мы можете найти любую информацию, связанную со сборкой, и любую информацию о настойках задачи (Task).

TaskResult

TaskResult позволяет вам хранить любую resultData для последующего использования 
и TaskState который будет указавать завершена задача со статусом Failed или Passed. 

Bamboo будет использовать этот TaskState для того чтобы
определить стоит ли продолжать и переходить к выполнению следующих задач.  
Вы должны использовать TaskResultBuilder для создания TaskResults.

TaskException

Если что-то пошло не так вы можеет выбросить TaskException.  
Однако подумайте о том, что вы можете выбросит это исключение
или же вы можете просто сообщить что задача завершилась со статусом Failing.


# Executing an external process

Видя, что выполнение внешнего процесса - это то, что, как мы думаем, многие из вас захотят сделать, мы создали несколько утилит, чтобы сделать это проще.

ProcessService (который может быть injected) может использоваться для запуска внешнего процесса. Он принимает в ExternalProcessBuilder, который вы можете заполнить всем необходимым для запуска вашей команды. Например:


```
ExternalProcess externalProcess = processService.executeProcess(
                    taskContext,
                    new ExternalProcessBuilder()
                            .workingDirectory(config.getWorkingDirectory())
                            .env(config.getExtraEnvironment())
                            .command(config.getCommandline())
            );

```

# Reading Logs

Мы добавили инфраструктуру для анализа build-логов на лету.
Смотрите документацию по [LogInterceptor](http://docs.atlassian.com/atlassian-bamboo/latest/com/atlassian/bamboo/build/logger/LogInterceptor.html?_ga=2.4052146.385607501.1576507130-1549702659.1570801910).
Вы можете добавить эти перехватчики to [LogInterceptorStack](http://docs.atlassian.com/atlassian-bamboo/latest/com/atlassian/bamboo/build/logger/LogInterceptorStack.html?_ga=2.36624290.385607501.1576507130-1549702659.1570801910) в 
[BuildLogger](http://docs.atlassian.com/atlassian-bamboo/latest/com/atlassian/bamboo/build/logger/BuildLogger.html?_ga=2.36624290.385607501.1576507130-1549702659.1570801910).
Примеры использования 
com.atlassian.bamboo.build.logger.interceptors.StringMatchingInterceptor and com.atlassian.bamboo.plugins.ant.task.AntBuildTask .


# Parsing Tests

Если вы хотите, чтобы ваша задача анализировала результаты теста, 
вы можете добавить TestCollationService. 
Он будет анализировать файлы сборки для тестов на основе переданного в filePattern. 
По умолчанию он анализирует отформатированные тесты JUnit, 
однако служба позволяет вам предоставить собственный 
TestReportCollector для поддержки других форматов.

# Configuring The Task

Если ваш TaskType требует указания настроек, то вы можете создать специальный настроечный класс.
Класс для создания настроек должен реализовывать [TaskConfigurator](http://docs.atlassian.com/atlassian-bamboo/latest/com/atlassian/bamboo/task/TaskConfigurator.html?_ga=2.234274588.385607501.1576507130-1549702659.1570801910).
Вы также можете сделать шаблон на основе freemarker 
для редактирования и просмотра настроек задачи.

ИЗОБРАЖЕНИЕ
https://developer.atlassian.com/server/bamboo/images/configuration-elements-highlighted-1.jpg

# Утилиты


AbstractTaskConfigurator Этот абстрактный класс
предоставляет пустые реализации методов интерфейса TaskConfigurator. 
Вы можете расширить этот класс, упростив таким образом свою задачу 
до реализации до более базовых требований.

TaskConfiguratorHelper 
TaskConfiguratorHelper может быть injected в ваш TaskConfigurator класс. 
Он содержит множество служебных методов для обработки базовых требований к настройкам, 
особенно перемешение настроек из TaskDefinition в FreemarkerContext 
и из ActionParameters обратно в TaskDefinition, например.


```
private static final List<String> FIELDS_TO_COPY = ImmutableList.of("goal", "environment");

    public Map<String, String> generateTaskConfigMap(@NotNull final ActionParametersMap params, @Nullable final TaskDefinition previousTaskDefinition)
    {
        final HashMap<String,String> config = Maps.newHashMap();
        taskConfiguratorHelper.populateTaskConfigMapWithActionParameters(config, params, FIELDS_TO_COPY);
        return config;
    }

    public void populateContextForEdit(@NotNull Map<String, Object> context, @NotNull TaskDefinition taskDefinition)
    {
        taskConfiguratorHelper.populateContextWithConfiguration(context, taskDefinition, FIELDS_TO_COPY);
    }

```

The TaskConfiguratorHelper также содержит утилиты,
для работы с некоторыми общими полями,
такими как выборк JDK, 
и установка директории в котором можно найти результаты тестов (Test Results). 

Однако эти методы, связаны со специальными named parameters, 
поэтому если вы хотите использовать эту функциональность
вы должны использовать TaskConfigConstants для именования полей.

# Tasks, Requirements And Capabilities (задачи, требования и возможности)

## Executable Capabilities
В вашем плагине, вы можете указать,
что определенный исполняемый файл досупен для использования,
например, Maven task требует чтобы 
exe Maven был на агенте,
чтобы иметь возможность запускать build команды Maven.
Чтобы ваш исполняемый файл был доступен в списке возможных исполняемых файлов,
которые можно добавить в качестве Capabilities,
необходимо определить исполняемый файл с помощью TaskType.

ИЗОБРАЖЕНИЕ
https://developer.atlassian.com/server/bamboo/images/executabledefinition.jpg


| Attribute | Details|
| ------------- |:-------------:|
| key|уникальный ключ этого исполняемого файла.This is prefixed with "system.builder" and the postfixed with the Executable Label provided by the user to create the key of the Capability |
| nameKey|i18n key for the display Name of the Executable Type.  Will be visible when configuring Capabilities. Required: no Default: Name of the Plugin Modul|
| pathHelpKey|i18n key for the help text to be displayed under the Path field for the capability.  Be clear what you expect the user to enter, e.g. the directory containing the exe, or the path of the actual exe Required: no Default: -|
| primaryCapabilityProvider|This field is only applicable if there is more than one TaskType module defining the same executable type.  If so, set to false.Required: no Default: True|

## Providing Default Capabilities
Если вы хотите, чтобы исполняемые файлы на Агенте или Сервере
автоматически добавлялись в качестве Capabilities,
вы можете использовать CapabilityDefaultsHelper

ИЗОБРАЖЕНИЕ
https://developer.atlassian.com/server/bamboo/images/capabilitydefaulthighlighted.jpg

Предоставляемый класс должен реализовывать интерфейс CapabilityDefaultsHelper,
Есть также некоторые абстрактные классы,
включая AbstractFileCapabilityDefaulstHelper
и AbstractHomeDirectoryCapabilityDefaultsHelper
Помните, что CapabilityDefaultHelpers 
должен быть способен к BAMBOO:run на удаленных агентах.

## Требования к задаче
Требования сейчас гораздо более тесто связаны с задачами.
Задача несет ответственность за обеспечение требований,
которые необходимо выполнить.
Если вы хотите, чтобы у ваших задач были требования к Capabilities
для ваших собственных исполняемых модулей,
определенных для плагина,
или для любых других Capabilities (таких как JDK),
вашему TaskConfigurator также нужно будет реализовать TaskRequirementSupport
Этот класс просто предоставляет метод для возврата списка требований
к задаче, который затему будет управляться внутри Bamboo.


### Исполняемые файлы в вашем пользовательском интерфейсе (UI)
Часто вы хотите, чтобы пользователи выбирали,
какой исполняемый файл или JDK они хотят использовать для этой задачи.
Вы можете использовать наш UIConfigBEan для заполнения списка
доступных исполняемых файлов JDK.
Затем вы можете использовать значение,
предоставленное пользователем,
чтобы создать правильный Capability key,
который будет испльзоваться при создании вашего требования.

```

[@ww.select cssClass="builderSelectWidget" labelKey='executable.type' name='selectedExecutable'
            list=uiConfigBean.getExecutableLabels('myExecutable') /]

[@ww.select cssClass="jdkSelectWidget"
            labelKey='builder.common.jdk' name='buildJdk'
            list=uiConfigBean.jdkLabels required='true' /]

```

### Создание требований
Итак, как создать правильные требования?
Для этого нужно создать новый RequirementImpl

```
public RequirementImpl(String key, boolean regexMatch, String match)
```


|Argument |Description|
| ------------- |:-------------:|
|key|The key of the Capability to require.; Example: system.builder.ant.Ant+1.8|
|regexMatch|True if it requires a regex match, false if it requires an exact match. Set to true, if you only need the capability to exist.; Example: true|
|match|The string to match to. Set to ".*" if you only need the capability to exist; Example: .*|
|||
|||


For Example:

```
final String executableLabel = taskDefinition.getConfiguration().get("selectedExecutable");
   Requirement req = new RequirementImpl("system.builder.myExecutable" + "." + executableLabel, true, ".*")

   final String jdkLabel = taskDefinition.getConfiguration().get("buildJdk");
   Requirement req = new RequirementImpl("system.builder.jdk" + "." + jdkLabel, true, ".*")
```

Utilities The TaskConfiguratorHelper provides some methods to assist in creating requirements including:

```java
void addJdkRequirement(@NotNull Set<Requirement> requirements, @NotNull TaskDefinition taskDefinition, @NotNull String cfgJdkLabel);

    void addSystemRequirementFromConfiguration(@NotNull Set<Requirement> requirements, @NotNull TaskDefinition taskDefinition,
                                               @NotNull String cfgKey, @NotNull String requirementPrefix);

```

## Running On Remote Agents
Throughout this overview there have been several mentions of things that must be able to run on Remote Agents, so, what does this mean? On a Remote Agent your code does not have access to the database or the server's file system. So this means things like extended configuration information or previous results are not available to you. It also means that a lot of Managers which you can usually use are off-limits. Apart from things passed into your classes, the following is the list of Services you are allowed to use on remote agents at the time of writing this. These can be injected into your plugin classes.

* TestCollationService
* ProcessService
* RemotableEventManager
* ArtifactManager
* CustomVariableContext
* ErrorUpdateHandler
* AgentContext
* CapabilityConfigurationManager
* CapabilityContext
* CapabilityDefaultsHelper
* SystemInfo

## Updating your Builder configuration
If you have migrated a Builder to Tasks, you can use the LegacyBuilderToTaskConverter Plugin Module to update the data.
