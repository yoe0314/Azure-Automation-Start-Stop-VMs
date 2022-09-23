
# Azure Automation Start/Stop VMs

###### tags:`Azure Automation`

[toc]

本篇內容將透過Azure Automation account的PowerShell Runbook來實作VM自動開關機。
## 1/ Create Automation account
* 於Azure Portal上方Search Bar搜尋`自動化帳戶`點選建立。
* 選擇要放在哪個資源群組、區域並給他一個名稱。
![](https://i.imgur.com/GzCzS4w.png)

## 2/ Use Azure Managed Identities
* 使用Azure身分受控識別，否則我們在後續建立PowerShell Runbook測試執行時會出現錯誤。
> [什麼是適用於 Azure 資源的受控識別？](https://learn.microsoft.com/zh-tw/azure/active-directory/managed-identities-azure-resources/overview)
* 點選`身分識別`，狀態要是`開啟`，並點選`Azure角色指派`。

![](https://i.imgur.com/c8nK6ab.png)
* 點選`+新增角色指派`。

![](https://i.imgur.com/LMcUP3f.png)
* `角色`選擇Contributor以上的權限。

![](https://i.imgur.com/MSSdLtC.png)
> [使用 Azure 入口網站指派 Azure 角色](https://learn.microsoft.com/zh-tw/azure/role-based-access-control/role-assignments-portal?tabs=current)
* 完成角色指派畫面如下。

![](https://i.imgur.com/xH0eYNv.png)

## 3/ Create Runbook
* 完成角色指派後，接著我們就可以來建立Runbook。
### Create Runbook
* 點選`Runbook`，`+建立Runbook`。

![](https://i.imgur.com/PDlfTsS.png)
* 給他一個名稱，例如：AutoStartVM。
* `Runbook類型`選擇PowerShell。
* `執行階段版本`選擇5.1。
* `說明`可輸入這個Runbook的用途說明。

![](https://i.imgur.com/eS1H0ts.png)
* 接著貼入下段PowerShell指令。
```powershell=
Param(
 [string]$VmName,
 [string]$ResourceGroupName,
 [ValidateSet("Start", "Stop")]
 [string]$VmAction
)

Connect-AzAccount -Identity

# Start Virtual Machines for example, VM01,VM02,VM03 etc
$vms = $VmName.split(',')
foreach($vm in $vms) {
IF ($VmAction -eq "Start") {
    Start-AzVM -Name $Vm -ResourceGroupName $ResourceGroupName | Out-Null
    #Write-Output "VM $Vm in Resource Group $ResourceGroupName was started Successfully" 
    $objOut = [PSCustomObject]@{
    ResourceGroupName = $ResourceGroupName
    VMName = $Vm
    VMAction = $VmAction
    }
    Write-Output ( $objOut | ConvertTo-Json)
    }
}

# Shutdown Virtual Machines for example, VM01,VM02,VM03 etc
$vms = $VmName.split(',')
foreach($vm in $vms) {
IF ($VmAction -eq "Stop") {
    Stop-AzVM -Name $Vm -ResourceGroupName $ResourceGroupName -Force | Out-Null
    #Write-Output "VM $Vm in Resource Group $ResourceGroupName was stopped Successfully"
    $objOut = [PSCustomObject]@{
    ResourceGroupName = $ResourceGroupName
    VMName = $Vm
    VMAction = $VmAction
    }
    Write-Output ( $objOut | ConvertTo-Json)
    }
}
```
* 點選`儲存`並且點選`測試窗格`來進行測試。
![](https://i.imgur.com/DwG1N0c.png)
* 輸入相對應的參數。
* `VMNAME`：可以輸入多台VM，用`,`隔開。
* `RESOURCEGROUPNAME`：VM所處的資源群組名稱。
* `VMACTION`：輸入開機`Start`或關機`Stop`。
* 填完後點選`啟動`來看VM是否有真的運行或關機成功，右方窗格會顯示執行成果。

![](https://i.imgur.com/EZCbR27.png)
* 實際查看VM也有成功開機。

![](https://i.imgur.com/tgzzfnu.png)
* 測試沒問題後我們就可以`發行`這個Runbook。

![](https://i.imgur.com/qivX9ja.png)

### Create Schedule
* 發行Runbook後，我們要來新增排程。
* 點選`排程`，`+加入排程`。

![](https://i.imgur.com/ubsx11u.png)
* 點選排程

![](https://i.imgur.com/jQxKgId.png)
* 給此排程一個名稱。
* 選擇開始的時間、週期和是否要有到期日。
![](https://i.imgur.com/rmLY0Lp.png)
* 點選`建立`。
* 接著設定`參數與回合設定`。

![](https://i.imgur.com/8wUpl2r.png)
* 輸入你所要排程的機器及相對應參數，並點選`確定`。

![](https://i.imgur.com/RWW2jRI.png)
* 這樣排程就完成了，這個Runbook就會在我們指定的時間和頻率自動執行這個PowerShell腳本。
* 排程設定完後大概會如下圖，我設定兩個排程，一個自動開機AutoStartVMs，一個自動關機AutoStopVMs。
![](https://i.imgur.com/GgXRmfa.png)
* 我們可以到`工作`來看我們設定的Runbook有沒有實際執行以及他的執行歷程。

![](https://i.imgur.com/yrSOxNU.png)

### Additional Information
* 其實要在Azure上達成VM的自動開關機有許多種方法，上述的PowerShell Runbook、Logic App等方法都可以達到目的。
* 這邊分享Will保哥的一篇Azure自動開關機的文章，操作方法非常簡單(實際上就是透過Logic App達成)。
> [如何設定 Azure 虛擬機器 (VM) 在早上自動開機、晚上自動關機 - Will 保哥](https://blog.miniasp.com/post/2021/06/12/Auto-Shutdown-and-Auto-Start-during-Off-hours-in-an-Azure-VM)
## Reference
* [什麼是適用於 Azure 資源的受控識別？](https://learn.microsoft.com/zh-tw/azure/active-directory/managed-identities-azure-resources/overview)
* [使用 Azure 入口網站指派 Azure 角色](https://learn.microsoft.com/zh-tw/azure/role-based-access-control/role-assignments-portal?tabs=current)
* [Start/Stop VMs during off-hours overview](https://learn.microsoft.com/en-us/azure/automation/automation-solution-vm-management?WT.mc_id=DT-MVP-4015686)
* [Azure Automation documentation](https://learn.microsoft.com/en-us/azure/automation/?WT.mc_id=DT-MVP-4015686)
* [Simple Stop & Start Azure VMs Powershell – Part 1/2](https://www.cloudinspired.com/simple-stop-start-azure-vms-powershell/)







