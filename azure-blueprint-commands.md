##Enable az cli blueprint extension
az extension add --name blueprint

#AzBluePrint with PowerShell

##Install the Azure Blueprints module from PowerShell Gallery
Install-Module -Name Az.Blueprint

##Login first with Connect-AzAccount if not using Cloud Shell

##Get a reference to the new blueprint object, we'll use it in subsequent steps
$blueprint = New-AzBlueprint -Name 'block-resource2' -BlueprintFile .\Blueprint.json

##Add ARM Template artifact
New-AzBlueprintArtifact -Blueprint $blueprint -Name 'Storage' -ArtifactFile ./Artifacts/62805c29-367e-4117-bf51-3dd8d5c9fa15.json

##Publish Blueprint new version
Publish-AzBlueprint -Blueprint $blueprint -Version '1.0'

##Assign Blueprint
New-AzBlueprintAssignment -Blueprint $blueprint -Name 'assignMyBlueprint' -AssignmentFile ./assignment_params.json


#References
https://docs.microsoft.com/en-us/azure/governance/blueprints/?WT.mc_id=blueprintsextension-github-nepeters

https://github.com/MicrosoftDocs/azure-docs/blob/master/articles/governance/blueprints/how-to/manage-assignments-ps.md

https://dev.azure.com/mrptsai/Blueprints

https://www.wesleyhaakman.org/azure-blueprints-level-parameters/

https://github.com/MicrosoftDocs/azure-docs/blob/master/articles/governance/blueprints/create-blueprint-powershell.md

https://github.com/MicrosoftLearning/AZ-500-Azure-Security/blob/master/Instructions/Labs/Module_2/LAB_20_Azure%20Blueprints.md

https://docs.microsoft.com/en-us/azure/governance/blueprints/concepts/parameters

https://github.com/Azure/azure-blueprints/tree/master/pipelines-scripts

https://docs.microsoft.com/en-us/cli/azure/ext/blueprint/blueprint?view=azure-cli-latest

https://docs.microsoft.com/en-us/powershell/module/az.blueprint/export-azblueprintwithartifact?view=azps-3.8.0

#PowerShell script

Install-Module -Name Az.DevOps.Blueprint -MinimumVersion 1.0.3 -Force -Scope CurrentUser -Verbose

Import-Module -Name Az.DevOps.Blueprint -MinimumVersion 1.0.3 -Force -Scope Local -Verbose

$latestBp = Get-AzBlueprint -Name $(BpName) -LatestPublished -ErrorAction SilentlyContinue

if($latestBp) {
	$blueprint = Set-AzBlueprint -Name '$(BpName)' -BlueprintFile block-resource\Blueprint.json
	Set-AzBlueprintArtifact -Blueprint $blueprint -Name 'Storage' -ArtifactFile block-resource/Artifacts/62805c29-367e-4117-bf51-3dd8d5c9fa15.json
} else {
	$blueprint = New-AzBlueprint -Name '$(BpName)' -BlueprintFile block-resource\Blueprint.json 
	New-AzBlueprintArtifact -Blueprint $blueprint -Name 'Storage' -ArtifactFile block-resource/Artifacts/62805c29-367e-4117-bf51-3dd8d5c9fa15.json
}

Publish-AzBlueprint -Blueprint $blueprint -Version 'Build-$(Build.BuildNumber)'

$existingAssignedBp = Get-AzBlueprintAssignment -Name "$(BpAssignmentName)" -ErrorAction SilentlyContinue

if($existingAssignedBp) {
	Set-AzBlueprintAssignment -Blueprint $blueprint -Name $(BpAssignmentName) -AssignmentFile block-resource/assignment_params.json
}
else {
	Write-Host "No existing blueprint assignment. Creating one..."
	New-AzBlueprintAssignment -Blueprint $blueprint -Name $(BpAssignmentName) -AssignmentFile block-resource/assignment_params.json
}
