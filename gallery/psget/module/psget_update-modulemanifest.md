---
ms.date: 2017-06-12
contributor: manikb
ms.topic: reference
keywords: "коллекция,powershell,командлет,psget"
title: Update-ModuleManifest
ms.openlocfilehash: ce3f6f173535d98648eb51adb1dbf84764e4f434
ms.sourcegitcommit: 75f70c7df01eea5e7a2c16f9a3ab1dd437a1f8fd
ms.translationtype: HT
ms.contentlocale: ru-RU
ms.lasthandoff: 06/12/2017
---
# <a name="update-modulemanifest"></a><span data-ttu-id="30747-103">Update-ModuleManifest</span><span class="sxs-lookup"><span data-stu-id="30747-103">Update-ModuleManifest</span></span>
<span data-ttu-id="30747-104">Обновляет файл манифеста модуля.</span><span class="sxs-lookup"><span data-stu-id="30747-104">Updates a module manifest file.</span></span>

## <a name="description"></a><span data-ttu-id="30747-105">Описание</span><span class="sxs-lookup"><span data-stu-id="30747-105">Description</span></span>

<span data-ttu-id="30747-106">Командлет Update-ModuleManifest обновляет файл манифеста модуля (PSD1).</span><span class="sxs-lookup"><span data-stu-id="30747-106">The Update-ModuleManifest cmdlet updates a module manifest (.psd1) file.</span></span>

### <a name="notes"></a><span data-ttu-id="30747-107">Заметки</span><span class="sxs-lookup"><span data-stu-id="30747-107">Notes</span></span>
    - <span data-ttu-id="30747-108">DscResourcesToExport поддерживается только в последней версии PowerShell, 5.0.</span><span class="sxs-lookup"><span data-stu-id="30747-108">DscResourcesToExport is only supported on the latest PowerShell version 5.0.</span></span> <span data-ttu-id="30747-109">При использовании предыдущей версии PowerShell обновить поле нельзя.</span><span class="sxs-lookup"><span data-stu-id="30747-109">We won’t be able to update the field if you are running on lower versions of PowerShell.</span></span>

## <a name="cmdlet-syntax"></a><span data-ttu-id="30747-110">Синтаксис командлета</span><span class="sxs-lookup"><span data-stu-id="30747-110">Cmdlet syntax</span></span>
```powershell
Get-Command -Name Update-ModuleManifest -Module PowerShellGet -Syntax
```

## <a name="cmdlet-online-help-reference"></a><span data-ttu-id="30747-111">Ссылка на раздел справки по командлету в Интернете</span><span class="sxs-lookup"><span data-stu-id="30747-111">Cmdlet online help reference</span></span>

[<span data-ttu-id="30747-112">Update-ModuleManifest</span><span class="sxs-lookup"><span data-stu-id="30747-112">Update-ModuleManifest</span></span>](http://go.microsoft.com/fwlink/?LinkId=619311)

## <a name="example-commands"></a><span data-ttu-id="30747-113">Примеры команд</span><span class="sxs-lookup"><span data-stu-id="30747-113">Example commands</span></span>

<span data-ttu-id="30747-114">Этот новый командлет используется для обновления файла манифеста с помощью входных значений свойств.</span><span class="sxs-lookup"><span data-stu-id="30747-114">This new cmdlet is used to help update manifest file with input property values.</span></span> <span data-ttu-id="30747-115">Он принимает все те же параметры, что и New-ModuleManifest.</span><span class="sxs-lookup"><span data-stu-id="30747-115">It takes all parameters that New-ModuleManifest does.</span></span>

<span data-ttu-id="30747-116">Нам известно, что многие создатели модулей стремятся указать "\*" в экспортированных значениях, таких как FunctionsToExport, CmdletsToExport и т. д. Во время публикации модуля в коллекции PowerShell неуказанные функции и команды будут занесены неправильно.</span><span class="sxs-lookup"><span data-stu-id="30747-116">We notice that a lot of module authors would like to specify “\*” in exported values such as FunctionsToExport, CmdletsToExport, etc. During module publishing to PowerShell Gallery, unspecified functions and commands will not be populated properly onto the Gallery.</span></span> <span data-ttu-id="30747-117">Поэтому мы рекомендуем авторам модулей обновить манифесты с использованием соответствующих значений.</span><span class="sxs-lookup"><span data-stu-id="30747-117">Therefore, we suggest module authors update their manifests with proper values.</span></span>

<span data-ttu-id="30747-118">При наличии модулей, имеющих экспортированные свойства, Update-ModuleManifest заполнит указанный файл манифеста данными из экспортированных функций, командлетов, переменных и т. п:</span><span class="sxs-lookup"><span data-stu-id="30747-118">If you have modules that have exported properties, Update-ModuleManifest will fill the specified manifest file with information from exported functions, cmdlets, variables etc:</span></span>
```powershell
Get-Content -Path "C:\Temp\PSGTEST-TestPackageMetadata\2.5\PSGTEST-TestPackageMetadata.psd1"
@{
# Script module or binary module file associated with this manifest.
# RootModule = ''
# Version number of this module.
ModuleVersion = '2.5'
# ID used to uniquely identify this module
GUID = '610e5c5b-dc42-4eaa-8511-ebfb44066d5e'

#(Other properties removed here for Simplicity…)

# Functions to export from this module
FunctionsToExport = '*'
# Cmdlets to export from this module
CmdletsToExport = '*'
# Variables to export from this module
VariablesToExport = '*'
# Aliases to export from this module
AliasesToExport = '*'
}
```

<span data-ttu-id="30747-119">После Update-ModuleManifest:</span><span class="sxs-lookup"><span data-stu-id="30747-119">After Update-ModuleManifest:</span></span>
```powershell
Update-ModuleManifest -Path "C:\Temp\PSGTEST-TestPackageMetadata\2.5\PSGTEST-TestPackageMetadata.psd1"
Get-Content -Path "C:\Temp\PSGTEST-TestPackageMetadata\2.5\PSGTEST-TestPackageMetadata.psd1"
#
# Module manifest for module 'NewManifest'
#
# Generated by: author name
#
# Generated on: 11/13/2015
#
@{
# Script module or binary module file associated with this manifest.
# RootModule = ''
# Version number of this module.
ModuleVersion = '2.5'
# ID used to uniquely identify this module
GUID = '610e5c5b-dc42-4eaa-8511-ebfb44066d5e'
# Functions to export from this module
FunctionsToExport = 'Get-FooFn Get-FooWF'
# Cmdlets to export from this module
CmdletsToExport = 'Test-PSGetTestCmdlet'
}
```

<span data-ttu-id="30747-120">Для каждого модуля имеются сопоставленные с ним поля метаданных.</span><span class="sxs-lookup"><span data-stu-id="30747-120">For each module, there are also metadata fields associated with it.</span></span> <span data-ttu-id="30747-121">Для правильного отображения метаданных в коллекции PowrShell можно воспользоваться Update-ModuleManifest, чтобы заполнить эти поля в PrivateData.</span><span class="sxs-lookup"><span data-stu-id="30747-121">In order to display metadata properly on PowrShell Gallery, you can use Update-ModuleManifest to populate those fields under PrivateData.</span></span>

```powershell
Update-ModuleManifest -Path "C:\Temp\PSGTEST-TestPackageMetadata\2.5\PSGTEST-TestPackageMetadata.psd1" -Tags "Tag1" -LicenseUri "http://license.com" -ProjectUri "http://project.com" -IconUri "http://icon.com" -ReleaseNotes "Test module"
```

<span data-ttu-id="30747-122">Хэш-таблица PrivateData из шаблона файла манифеста имеет следующие свойства.</span><span class="sxs-lookup"><span data-stu-id="30747-122">PrivateData hashtable from the manifest file template has the following properties</span></span>

```powershell
# Private data to pass to the module specified in RootModule/ModuleToProcess. This may also contain a PSData hashtable with additional module metadata used by PowerShell.
PrivateData = @{
    PSData = @{
        # Tags applied to this module. These help with module discovery in online galleries.
        # Tags = @()

        # A URL to the license for this module.
        # LicenseUri = ''
    
        # A URL to the main website for this project.
        # ProjectUri = ''
        
        # A URL to an icon representing this module.
        # IconUri = ''
        
        # ReleaseNotes of this module
        # ReleaseNotes = ''
        
        # External dependent modules of this module
        # ExternalModuleDependencies = ''
    } # End of PSData hashtable
} # End of PrivateData hashtable
```
