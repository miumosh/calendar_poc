# Calendar POC

## Teams グループのメール・予定表の表示設定 有効化
[365 グループと Teams の関連性](https://learn.microsoft.com/ja-jp/MicrosoftTeams/office-365-groups)

### 事前準備
- [Micrsoft365管理センター](https://admin.microsoft.com/)へアクセス
- 役割-役割の割り当て＞Exchangeタブ＞Recipient Management＞割り当て済みタブ＞作業ユーザーを追加
- 役割-役割の割り当て＞Exchangeタブ＞View-Only Organization Management＞割り当て済みタブ＞作業ユーザーを追加


### 設定変更

- ツール導入
```Powershell
# Powershell を管理者として実行

# 必要であれば、実行ポリシーの変更
Get-ExecutionPolicy
Set-ExecutionPolicy RemoteSigned

# Exchange モジュール利用準備
Get-InstalledModule -Name ExchangeOnlineManagement
Install-Module -Name ExchangeOnlineManagement
Update-Module -Name ExchangeOnlineManagement
Import-Module ExchangeOnlineManagement

# Exchange Online 接続（GUI ログイン画面に遷移するので、権限を持ったユーザーでログイン）
Connect-ExchangeOnline
```

- 設定変更
```Powershell
# Exchange Online 接続（GUI ログイン画面に遷移するので、権限を持ったユーザーでログイン）
Connect-ExchangeOnline

# 対象ドメインへの接続を確認
Get-AcceptedDomain
# Name                           DomainName                     DomainType     Default
# ----                           ----------                     ----------     -------
# <ms_domain>.onmicrosoft.com   <ms_domain>.onmicrosoft.com   Authoritative  True


# 自身のユーザーID を確認
Get-Recipient -Identity "user_01@<ms_domain>.onmicrosoft.com"
# Name                                 RecipientType
# ----                                 -------------
# 19df7347-xxxx-xxxx-xxxx-xxxxxxxxxxxx UserMailbox

# 役割グループの所属ユーザーを確認
Get-RoleGroupMember "Recipient Management"
# Name                                 RecipientType
# ----                                 -------------
# 19df7347-xxxx-xxxx-xxxx-xxxxxxxxxxxx UserMailbox

Get-RoleGroupMember "View-Only Organization Management"
# Name                                 RecipientType
# ----                                 -------------
# 19df7347-xxxx-xxxx-xxxx-xxxxxxxxxxxx UserMailbox
# HelpdeskAdmins_-xxxxxxxxxx           Group
# GlobalReaders_-xxxxxxxxxx            Group


# 自身に割当たってる権限を確認（上記で確認できてるので参考程度）
Get-ManagementRoleAssignment -RoleAssignee "user_01@<ms_domain>.onmicrosoft.com"
# Name                           Role                            RoleAssigneeName                RoleAssigneeType                AssignmentMethod               EffectiveUserName
# ----                           ----                            ----------------                ----------------                ----------------               -----------------
# My Custom Apps-Default Role... My Custom Apps                  Default Role Assignment Policy  RoleAssignmentPolicy            RoleAssignmentPolicy           すべてのポリシー割り当て先
# My Marketplace Apps-Default... My Marketplace Apps             Default Role Assignment Policy  RoleAssignmentPolicy            RoleAssignmentPolicy           すべてのポリシー割り当て先
# My ReadWriteMailbox Apps-De... My ReadWriteMailbox Apps        Default Role Assignment Policy  RoleAssignmentPolicy            RoleAssignmentPolicy           すべてのポリシー割り当て先
# MyBaseOptions-Default Role ... MyBaseOptions                   Default Role Assignment Policy  RoleAssignmentPolicy            RoleAssignmentPolicy           すべてのポリシー割り当て先
# MyContactInformation-Defaul... MyContactInformation            Default Role Assignment Policy  RoleAssignmentPolicy            RoleAssignmentPolicy           すべてのポリシー割り当て先
# MyMailSubscriptions-Default... MyMailSubscriptions             Default Role Assignment Policy  RoleAssignmentPolicy            RoleAssignmentPolicy           すべてのポリシー割り当て先
# MyProfileInformation-Defaul... MyProfileInformation            Default Role Assignment Policy  RoleAssignmentPolicy            RoleAssignmentPolicy           すべてのポリシー割り当て先
# MyRetentionPolicies-Default... MyRetentionPolicies             Default Role Assignment Policy  RoleAssignmentPolicy            RoleAssignmentPolicy           すべてのポリシー割り当て先
# MyTextMessaging-Default Rol... MyTextMessaging                 Default Role Assignment Policy  RoleAssignmentPolicy            RoleAssignmentPolicy           すべてのポリシー割り当て先
# MyVoiceMail-Default Role As... MyVoiceMail                     Default Role Assignment Policy  RoleAssignmentPolicy            RoleAssignmentPolicy           すべてのポリシー割り当て先
# MyDistributionGroupMembersh... MyDistributionGroupMembership   Default Role Assignment Policy  RoleAssignmentPolicy            RoleAssignmentPolicy           すべてのポリシー割り当て先
# MyDistributionGroups-Defaul... MyDistributionGroups            Default Role Assignment Policy  RoleAssignmentPolicy            RoleAssignmentPolicy           すべてのポリシー割り当て先
# Distribution Groups-Recipie... Distribution Groups             Recipient Management            RoleGroup                       RoleGroup                      すべてのグループ メンバー
# Mail Recipient Creation-Rec... Mail Recipient Creation         Recipient Management            RoleGroup                       RoleGroup                      すべてのグループ メンバー
# Mail Recipients-Recipient M... Mail Recipients                 Recipient Management            RoleGroup                       RoleGroup                      すべてのグループ メンバー
# Message Tracking-Recipient ... Message Tracking                Recipient Management            RoleGroup                       RoleGroup                      すべてのグループ メンバー
# Migration-Recipient Management Migration                       Recipient Management            RoleGroup                       RoleGroup                      すべてのグループ メンバー
# Move Mailboxes-Recipient Ma... Move Mailboxes                  Recipient Management            RoleGroup                       RoleGroup                      すべてのグループ メンバー
# Recipient Policies-Recipien... Recipient Policies              Recipient Management            RoleGroup                       RoleGroup                      すべてのグループ メンバー
# Reset Password-Recipient Ma... Reset Password                  Recipient Management            RoleGroup                       RoleGroup                      すべてのグループ メンバー
# View-Only Configuration-Vie... View-Only Configuration         View-Only Organization Manag... RoleGroup                       RoleGroup                      すべてのグループ メンバー
# View-Only Recipients-View-O... View-Only Recipients            View-Only Organization Manag... RoleGroup                       RoleGroup                      すべてのグループ メンバー


# 設定変更対象の既存設定を確認
$target_group = "team01"
Get-UnifiedGroup -Identity "$target_group" | select DisplayName,ResourceProvisioningOptions,Language,HiddenFromExchangeClientsEnabled
# DisplayName ResourceProvisioningOptions Language HiddenFromExchangeClientsEnabled
# ----------- --------------------------- -------- --------------------------------
# team01      {Team}                      en-US                                True (※ この値が True だと予定表利用できない)

# 設定を変更
Set-UnifiedGroup -Identity "$target_group" -HiddenFromExchangeClientsEnabled:$false
Get-UnifiedGroup -Identity "$target_group" | select DisplayName,ResourceProvisioningOptions,Language,HiddenFromExchangeClientsEnabled
# DisplayName ResourceProvisioningOptions Language HiddenFromExchangeClientsEnabled
# ----------- --------------------------- -------- --------------------------------
# team01      {Team}                      en-US                               False (※ False に変更されたことを確認)


# Exchange への接続を解除
Disconnect-ExchangeOnline
# 必要であれば、実行ポリシーの設定切り戻し
Set-ExecutionPolicy Restricted
```
