# apim-devops-production
API Management の DevOps 検証レポジトリ（本番環境用）

## create dev env

```powershell
$location = 'japaneast'
$devrg = 'apim-demo-dev-rg'
$devApimName = 'ainaba-apim-dev'
$strAccName = 'ainabaapimstr'

New-AzResourceGroup -Name $devrg -Location $location
New-AzResourceGroupDeployment -Name "apim-devenv" -ResourceGroupName $devrg `
    -TemplateFile .\arm-devenv.json -location $location -storageAccountName $strAccName -apimServiceName $devApimName

```

## extract definitions from devenv

[Azure DevOps resource kit](https://github.com/Azure/azure-api-management-devops-resource-kit)

```powershell

# download resource kit
$reskitVersion = '1.0.0-beta.4'
$reskiturl = "https://github.com/Azure/azure-api-management-devops-resource-kit/releases/download/$($reskitVersion)/reskit-$($reskitVersion).zip"
$zipfile = "reskit.zip"
Invoke-WebRequest -Uri $reskiturl -OutFile $zipfile
Expand-Archive $zipfile
Remove-Item $zipfile

# extract from api management dev instance
$config = [System.IO.Path]::GetFullPath(".\extractSettigs.json")
$output = [System.IO.Path]::GetFullPath(".\extractor\")
.\reskit\ArmTemplates.exe extract --extractorConfig $config --fileFolder $output

# revision
# current revision = false

```

## deploy to production

```powershell
$devrg = 'apim-demo-dev-rg'
$strAccName = 'ainabaapimstr'
$prodrg = "apim-demo-prod-rg"

# create blob container
$strAccKey = (Get-AzStorageAccountKey -ResourceGroupName $rg -Name $strAccName)[0].Value
$strctx = New-AzStorageContext -StorageAccountName $strAccName -StorageAccountKey $strAccKey

# upload blob
# https://docs.microsoft.com/ja-jp/azure/storage/blobs/storage-quickstart-blobs-powershell
$containerName = [DateTime]::Now.ToString("yyyyMMdd-HHmmss")
$container = New-AzStorageContainer -Context $strctx -Name $containerName
Get-ChildItem $output -Recurse | where {$_ -is [System.IO.FileInfo]} | foreach {
    $blob = [System.IO.Path]::GetRelativePath($output, $_.Fullname)
    Set-AzStorageBlobContent -File $_.Fullname -Container $containerName -Blob $blob -Context $strctx 
}

# generate sas
# https://docs.microsoft.com/ja-jp/azure/storage/common/storage-sas-overview
[DateTime]$exp = [DateTime]::UtcNow.AddDays(1)
$sastoken = New-AzStorageContainerSASToken -Context $strctx -Name $containerName -Permission racwtl -ExpiryTime $exp


# deployment
New-AzResourceGroup -Name $devrg -Location $location
# https://docs.microsoft.com/ja-jp/azure/azure-resource-manager/templates/deploy-powershell
$storageBaseUrl = "https://$($strAccName).blob.core.windows.net/$($containerName)/"
New-AzResourceGroupDeployment -Name $containerName -ResourceGroupName $prodrg `
    -TemplateFile .\arm-master.json -TemplateParameterFile .\arm-parameters.json  `
    -storageBaseUrl $storageBaseUrl -storageSasToken $sastoken -targetApiRevision 1

```