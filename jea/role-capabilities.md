---
ms.date: 2017-06-12
author: rpsqrd
ms.topic: conceptual
keywords: "jea,powershell,безопасность"
title: "Возможности ролей JEA"
ms.openlocfilehash: 10f5f390daccbb012be6ee7272041e777810ee12
ms.sourcegitcommit: 75f70c7df01eea5e7a2c16f9a3ab1dd437a1f8fd
ms.translationtype: HT
ms.contentlocale: ru-RU
ms.lasthandoff: 06/12/2017
---
# <a name="jea-role-capabilities"></a>Возможности ролей JEA

> Область применения: Windows PowerShell 5.0

При создании конечной точки JEA потребуется определить одну или несколько "возможностей ролей", описывающих, *что именно* пользователь может сделать в сеансе JEA.
Возможность роли — это файл данных PowerShell с расширением PSRC, содержащий все командлеты, функции, поставщики и внешние программы, которые должны быть доступны для подключающихся пользователей.

В этом разделе описывается создание файла возможностей ролей PowerShell для пользователей JEA.

## <a name="determine-which-commands-to-allow"></a>Определение разрешаемых команд

Прежде всего, при создании файла возможности роли нужно определить, к чему именно потребуется доступ пользователям, которым назначена эта роль.
Этот процесс сбора требований может занять некоторое время, однако он крайне важен.
Предоставление пользователям доступа к слишком малому набору командлетов и функций может помешать их нормальной работе.
Разрешение доступа к слишком большому числу командлетов и функций может предоставить пользователям более широкие возможности, чем подразумевалось вами, что ослабит защиту вашей системы.

То, как вы будете это делать, зависит от вашей организации и целей, однако приведенные ниже советы помогут придерживаться правильного направления.

1. **Определите**, какие команды пользователи применяют для выполнения поставленных задач. Для этого можно опросить ИТ-специалистов, проверить сценарии автоматизации либо проанализировать журналы или записи сеансов PowerShell.
2. По возможности **замените** используемые программы командной строки на эквиваленты PowerShell, чтобы сделать аудит и настройку JEA максимально удобными. Внешние программы не поддерживают такую детальную настройку ограничений, как собственные командлеты PowerShell и функции в JEA.
3. При необходимости **ограничьте** область действия командлетов, разрешив только отдельные параметры и их значения. Это особенно важно в том случае, если пользователям нужно разрешить управлять лишь частью системы.
4. **Создайте** настраиваемые функции на замену сложным командам или командам, которые трудно ограничить в JEA. Простая функция, включающая в себя сложную команду или применяющая дополнительную логику проверки, повышает уровень контроля и упрощает работу администраторов и конечных пользователей.
5. **Проверьте** полученный список допустимых команд на соответствие потребностям пользователей или служб автоматизации и при необходимости скорректируйте его.

Важно помнить, что команды в сеансе JEA часто запускаются с правами администратора (или иными повышенными правами).
Важно тщательно отбирать доступные команды, чтобы конечная точка JEA не позволяла подключающемуся пользователю повысить свои права.
Ниже приведено несколько примеров команд, которые в состоянии без ограничений быть использованы злоумышленниками.
Обратите внимание, что этот список не является исчерпывающим и служит лишь отправной точкой для дальнейших действий.

### <a name="examples-of-potentially-dangerous-commands"></a>Примеры потенциально опасных команд

Риск | Пример | Связанные команды
-----|---------|-----------------
Предоставление прав администратора подключающемуся пользователю для обхода JEA | `Add-LocalGroupMember -Member 'CONTOSO\jdoe' -Group 'Administrators'` | `Add-ADGroupMember`, `Add-LocalGroupMember`, `net.exe`, `dsadd.exe`
Выполнение произвольного кода, например вредоносных программ, эксплойтов или пользовательских сценариев для обхода защиты | `Start-Process -FilePath '\\san\share\malware.exe'` | `Start-Process`, `New-Service`, `Invoke-Item`, `Invoke-WmiMethod`, `Invoke-CimMethod`, `Invoke-Expression`, `Invoke-Command`, `New-ScheduledTask`, `Register-ScheduledJob`

## <a name="create-a-role-capability-file"></a>Создание файла возможностей ролей

Вы можете создать новый файл возможности роли с помощью командлета [New-PSRoleCapabilityFile](https://msdn.microsoft.com/powershell/reference/5.1/microsoft.powershell.core/New-PSRoleCapabilityFile).

```powershell
New-PSRoleCapabilityFile -Path .\MyFirstJEARole.psrc
```

Полученный файл можно открыть в текстовом редакторе и изменить, чтобы разрешить нужные команды для роли.
Справочная документация PowerShell содержит несколько примеров того, как можно настроить такой файл.

### <a name="allowing-powershell-cmdlets-and-functions"></a>Разрешение функций и командлетов PowerShell

Чтобы разрешить пользователям запускать функции или командлеты PowerShell, добавьте имя функции или командлета в поле VisibleFunctions или VisbibleCmdlets.
Если вы не уверены, является ли команда командлетом или функцией, выполните `Get-Command <name>` и проверьте свойство CommandType в выходных данных.

```powershell
VisibleCmdlets = 'Restart-Computer', 'Get-NetIPAddress'
```

Иногда область действия командлета или функции слишком широка для потребностей пользователей.
Например, администратору DNS, скорее всего, потребуется только доступ для перезапуска службы DNS.
В мультитенантной среде, где клиентам предоставляется доступ к средствам самостоятельного управления, эти клиенты должны быть в состоянии управлять только собственными ресурсами.
В этих случаях можно ограничить доступные параметры из командлета или функции.

```powershell

VisibleCmdlets = @{ Name = 'Restart-Computer'; Parameters = @{ Name = 'Name' }}

```

В более сложных сценариях может потребоваться ограничить значения, которые можно задать для этих параметров.
Возможности ролей позволяют задать набор допустимых значений или шаблон регулярного выражения, по которому оценивается, разрешены ли указанные входные данные.

```powershell
VisibleCmdlets = @{ Name = 'Restart-Service'; Parameters = @{ Name = 'Name'; ValidateSet = 'Dns', 'Spooler' }},
                 @{ Name = 'Start-Website'; Parameters = @{ Name = 'Name'; ValidatePattern = 'HR_*' }}
```

> [!NOTE]
> [Общие параметры PowerShell](https://technet.microsoft.com/en-us/library/hh847884.aspx) всегда разрешены, даже если ограничить доступные параметры.
> Их не следует явно указывать в поле "Parameters".

В следующей таблице описаны различные способы настройки видимого командлета или видимой функции.
Вы можете комбинировать любые приведенные ниже примеры в поле "VisibleCmdlets".

Пример                                                                                      | Вариант использования
---------------------------------------------------------------------------------------------|----------------------------------------------------------------------------------------------------------------------------------
`'My-Func'` или `@{ Name = 'My-Func' }`                                                       | Разрешает пользователю выполнить `My-Func` без каких-либо ограничений для параметров.
`'MyModule\My-Func'`                                                                         | Разрешает пользователю выполнить `My-Func` из модуля `MyModule` без каких-либо ограничений для параметров.
`'My-*'`                                                                                     | Разрешает пользователю выполнить любой командлет или функцию с командой `My`.
`'*-Func'`                                                                                   | Разрешает пользователю выполнить любой командлет или функцию с существительным `Func`.
`@{ Name = 'My-Func'; Parameters = @{ Name = 'Param1'}, @{ Name = 'Param2' }}`               | Разрешает пользователю выполнить `My-Func` с параметрами `Param1` и (или) `Param2`. Для параметров можно задать любое значение.
`@{ Name = 'My-Func'; Parameters = @{ Name = 'Param1'; ValidateSet = 'Value1', 'Value2' }}`  | Разрешает пользователю выполнить `My-Func` с параметром `Param1`. Для параметра можно задать только значение "Value1" или "Value2".
`@{ Name = 'My-Func'; Parameters = @{ Name = 'Param1'; ValidatePattern = 'contoso.*' }}`     | Разрешает пользователю выполнить `My-Func` с параметром `Param1`. Для параметра можно задать любое значение, начинающееся с "contoso".


> [!WARNING]
> Для обеспечения наилучшей безопасности при определении видимых командлетов или функций рекомендуется не использовать подстановочные знаки.
> Вместо этого следует явно указать каждую надежную команду, чтобы убедиться, что не будут случайно разрешены никакие другие команды, которые используют такую же схему именования.

Вы не можете применить ValidatePattern и ValidateSet для одного командлета или одной функции.

В противном случае ValidatePattern переопределит ValidateSet.

Дополнительные сведения о ValidatePattern см. в [этой записи блога *Hey, Scripting Guy!*](https://blogs.technet.microsoft.com/heyscriptingguy/2011/01/11/validate-powershell-parameters-before-running-the-script/) и справочных материалах о [регулярных выражениях PowerShell](https://technet.microsoft.com/en-us/library/hh847880.aspx).

### <a name="allowing-external-commands-and-powershell-scripts"></a>Разрешение внешних команд и сценариев PowerShell

Чтобы разрешить пользователям запускать исполняемые файлы и сценарии PowerShell (PS1) в сеансе JEA, нужно добавить полный путь к каждой программы в поле "VisibleExternalCommands".

```powershell
VisibleExternalCommands = 'C:\Windows\System32\whoami.exe', 'C:\Program Files\Contoso\Scripts\UpdateITSoftware.ps1'
```

Рекомендуется по возможности использовать командлеты или функции PowerShell, эквивалентные любым внешним исполняемым файлам, которые вы разрешаете, так как для этих функций и командлетов PowerShell вы можете управлять разрешенными параметрами.

Многие исполняемые файлы дают возможность считать текущее состояние и затем изменить его, просто указав различные параметры.

Например, рассмотрим роль администратора файлового сервера, который хочет проверить, какие сетевые папки размещаются на локальном компьютере.
Один из способов проверки заключается в использовании `net share`.
Однако разрешение использования net.exe очень опасно, так как администратор может легко воспользоваться этой командой для получения прав администратора с помощью `net group Administrators unprivilegedjeauser /add`.
Лучше разрешить командлет [Get-SmbShare](https://technet.microsoft.com/en-us/library/jj635704.aspx), которой дает тот же результат, но имеет гораздо более ограниченную область действия.

Делая внешние команды доступными для пользователей в сеансе JEA, всегда указывайте полный путь к исполняемому файлу, чтобы вместо него не запускалась другая (и потенциально вредоносная) программа с таким же именем, размещенная в другом месте.

### <a name="allowing-access-to-powershell-providers"></a>Разрешение доступа к поставщикам PowerShell

По умолчанию в сеансах JEA нет доступных поставщиков PowerShell.

Это в первую очередь служит для снижения риска раскрытия конфиденциальной информации и параметров конфигурации подключающемуся пользователю.

При необходимости доступ к поставщикам PowerShell можно разрешить с помощью команды `VisibleProviders`.
Для просмотра полного списка поставщиков запустите `Get-PSProvider`.

```powershell
VisibleProviders = 'Registry'
```

Для простых задач, которым требуется доступ к файловой системе, реестру, хранилищу сертификатов, или других конфиденциальных поставщиков можно также написать настраиваемую функцию, которая работает с поставщиком от имени пользователя.
К функциям, командлетам и внешним программам, доступным в сеансе JEA, не применяются те же ограничения, что и для JEA, то есть они могут по умолчанию получить доступ к любому поставщику.
В случаях, когда требуется скопировать файлы из конечной точки JEA или на нее, рассмотрите возможность использования [диска пользователя](session-configurations.md#user-drive).

### <a name="creating-custom-functions"></a>Создание настраиваемых функций

В файле возможностей роли можно создать настраиваемые функции, чтобы упростить выполнение сложных задач для конечных пользователей.
Настраиваемые функции также удобны, когда требуется расширенная логика проверки для значений параметров командлета.
Вы можете составлять простые функции в поле **FunctionDefinitions**:

```powershell
VisibleFunctions = 'Get-TopProcess'

FunctionDefinitions = @{
    Name = 'Get-TopProcess'

    ScriptBlock = {
        param($Count = 10)

        Get-Process | Sort-Object -Property CPU -Descending | Microsoft.PowerShell.Utility\Select-Object -First $Count
    }
}
```

> [!IMPORTANT]
> Не забудьте добавить имя настраиваемых функций в поле **VisibleFunctions**, чтобы пользователи JEA могли запускать их.


Основная часть (блок сценария) настраиваемых функций запускается в языковом режиме по умолчанию для данной системы, и на нее не налагаются ограничения языка JEA.
Это означает, что функции имеют доступ к файловой системе и реестру и могут запускать команды, которые не были сделаны видимыми в файле возможностей роли.
Примите меры, чтобы избежать выполнения произвольного кода при использовании параметров и передачи вводимых пользователем данных напрямую в такие командлеты, как `Invoke-Expression`.

В приведенном выше примере можно заметить, что полное имя модуля (FQMN) `Microsoft.PowerShell.Utility\Select-Object` было использовано вместо сокращенного `Select-Object`.
Функции, определенные в файлах возможностей роли, по-прежнему подчиняются области действия сеансов JEA, что включает в себя функции прокси-сервера, создаваемые JEA для ограничения существующих команд.

По умолчанию Select-Object является ограниченным командлетом во всех сеансах JEA, который не позволяет выбрать произвольные свойства для объектов.
Чтобы использовать Select-Object без ограничений в функциях, требуется явно запросить полную реализацию, указав полное имя модуля.
Любой ограниченный командлет в сеансе JEA будет демонстрировать одинаковое поведение при вызове из функции в соответствии с [приоритетом команд](https://msdn.microsoft.com/en-us/powershell/reference/3.0/microsoft.powershell.core/about/about_command_precedence) PowerShell.

При наличии множества настраиваемых функций может быть проще поместить их в [модуль сценария PowerShell](https://msdn.microsoft.com/en-us/library/dd878340(v=vs.85).aspx).
Затем можно сделать эти функции видимыми в сеансе JEA с помощью поля "VisibleFunctions", как и в случае со встроенными и сторонними модулями.

## <a name="place-role-capabilities-in-a-module"></a>Помещение возможностей ролей в модуль

Чтобы среда PowerShell обнаружила файл возможности ролей, его следует поместить в папку "RoleCapabilities" в модуле PowerShell.
Этот модуль может храниться в любой папке, включенной в переменную среды `$env:PSModulePath`, однако не следует помещать его в папку System32 (которая зарезервирована для встроенных модулей) либо в папку, где не являющиеся доверенными подключающиеся пользователи могут изменить его.
Ниже приведен пример создания базового модуля сценария PowerShell с именем *ContosoJEA* в папке "Program Files".

```powershell
# Create a folder for the module
$modulePath = Join-Path $env:ProgramFiles "WindowsPowerShell\Modules\ContosoJEA"
New-Item -ItemType Directory -Path $modulePath

# Create an empty script module and module manifest. At least one file in the module folder must have the same name as the folder itself.
New-Item -ItemType File -Path (Join-Path $modulePath "ContosoJEAFunctions.psm1")
New-ModuleManifest -Path (Join-Path $modulePath "ContosoJEA.psd1") -RootModule "ContosoJEAFunctions.psm1"

# Create the RoleCapabilities folder and copy in the PSRC file
$rcFolder = Join-Path $modulePath "RoleCapabilities"
New-Item -ItemType Directory $rcFolder
Copy-Item -Path .\MyFirstJEARole.psrc -Destination $rcFolder
```

В разделе [Основные сведения о модуле PowerShell](https://msdn.microsoft.com/en-us/library/dd878324.aspx) приведены дополнительные сведения о модулях PowerShell, манифестах модуля и переменной среды PSModulePath.

## <a name="updating-role-capabilities"></a>Изменение возможностей ролей


Файл возможностей ролей можно изменить в любое время, просто сохранив изменения в нем.
Все новые сеансы JEA, запущенные после изменения возможности роли, будут использовать обновленные возможности.

Именно поэтому так важно управлять доступом к папке возможностей роли.
Изменять файлы возможностей ролей должно быть разрешено только наиболее надежным администраторам.
Если пользователь, не являющийся доверенным, может изменять файлы возможностей ролей, он легко сможет получить доступ к командлетам, повышающим его права.


Администраторам, желающим заблокировать доступ к возможностям ролей, следует убедиться, что учетная запись Local System имеет доступ на чтение к файлам возможностей ролей и содержащим их модулям.

## <a name="how-role-capabilities-are-merged"></a>Способ слияния возможностей ролей

При входе в сеанс JEA пользователям может предоставляться доступ к нескольким возможностям ролей в зависимости от сопоставлений ролей в [файле конфигурации сеанса](session-configurations.md).
В этом случае JEA пытается предоставить такому пользователю *наиболее широкий* набор команд, разрешенных для любой из этих ролей.

**VisibleCmdlets и VisibleFunctions**

Наиболее сложная логика слияния затрагивает командлеты и функции, параметры и значения параметров которых могут быть ограничены в JEA.

Действуют следующие правила:

1. Если командлет является видимым всего в одной роли, он отображается для пользователя с любыми применимыми ограничениями параметров.
2. Если командлет является видимым в нескольких ролях, каждая из которых налагает на него схожие ограничения, он отображается пользователю с такими ограничениями.
3. Если командлет является видимым в нескольких ролях, каждая из которых разрешает иной набор параметров, пользователю отображается как командлет, так и все параметры, определенные для каждой роли. Если одна роль не имеет ограничений для параметров, будет разрешены все параметры.
4. Если одна роль определяет набор или шаблон проверок для параметра командлета, а другая разрешает параметр, но не ограничивает его значения, набор или шаблон проверок будет игнорироваться.
5. Если набор проверок определен для одного параметра командлета в нескольких ролях, все значения из всех наборов проверок будут разрешены.
6. Если шаблон проверок определен для одного параметра командлета в нескольких ролях, будут разрешены все значения, соответствующие любому из шаблонов.
7. Если набор проверок определен в одной или нескольких ролях и шаблон проверок определен в другой роли для того же параметра командлета, набор проверок игнорируется, а к оставшимся шаблонам проверок применяется правило (6).

Ниже приведен пример объединения ролей согласно этим правилам:

```powershell
# Role A Visible Cmdlets
$roleA = @{
    VisibleCmdlets = 'Get-Service',
                     @{ Name = 'Restart-Service'; Parameters = @{ Name = 'DisplayName'; ValidateSet = 'DNS Client' } }
}

# Role B Visible Cmdlets
$roleB = @{
    VisibleCmdlets = @{ Name = 'Get-Service'; Parameters = @{ Name = 'DisplayName'; ValidatePattern = 'DNS.*' } },
                     @{ Name = 'Restart-Service'; Parameters = @{ Name = 'DisplayName'; ValidateSet = 'DNS Server' } }
}

# Resulting permisisons for a user who belongs to both role A and B
# - The constraint in role B for the DisplayName parameter on Get-Service is ignored becuase of rule #4
# - The ValidateSets for Restart-Service are merged because both roles use ValidateSet on the same parameter per rule #5
$mergedAandB = @{
    VisibleCmdlets = 'Get-Service',
                     @{ Name = 'Restart-Service'; Parameters = @{ Name = 'DisplayName'; ValidateSet = 'DNS Client', 'DNS Server' } }
}
```



**VisibleExternalCommands, VisibleAliases, VisibleProviders, ScriptsToProcess**

Все другие поля в файле роли возможностей ролей просто добавляются в совокупный набор допустимых внешних команд, псевдонимов, поставщиков и сценариев запуска.
Команда, псевдоним, поставщик или сценарий, доступные в одной возможности роли, будут доступны пользователю JEA.

Внимательно следите за тем, чтобы комбинированный набор поставщиков из одной возможности роли и командлеты, функции и команды из другой не разрешали подключающимся пользователям непреднамеренный доступ к системным ресурсам.
Например, если одна роль разрешает командлет `Remove-Item`, а другая разрешает поставщик `FileSystem`, существует риск того, что пользователь JEA удалит произвольные файлы на компьютере.
Дополнительные сведения об определении действующих разрешений пользователей см. в [разделе об аудите JEA](audit-and-report.md).

## <a name="next-steps"></a>Дальнейшие действия

- [Создание файла конфигурации сеанса](session-configurations.md)

