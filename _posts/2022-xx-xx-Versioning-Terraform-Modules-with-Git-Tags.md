########### SEMVER
- script: |
    git config --global user.name "TerraformBuildService"
    git config --global user.email "terraform@boqdev.com.au"
    git tag $(Build.BuildId)
    git push origin 0.0.$(Build.BuildId)

########### SEMVER
- job: SemanticVersion
  steps:
  - task: GitVersion@5  
    displayName: 'Determine version'
    inputs:
      runtime: 'core'
      useConfigFile: true
      configFilePath: 'GitVersion.yml'
  - powershell: Write-Host "##vso[task.setvariable variable=NuGetVersion;isOutput=true]$env:NuGetVersion"
    name: GitVersion
    env:
      NuGetVersion: $(GitVersion.NuGetVersion)
  - powershell: 'dir env:'

########### SEMVER
- task: GitVersion@5  
displayName: 'Determine version'
inputs:
  runtime: 'core'
  useConfigFile: true
  configFilePath: 'GitVersion.yml'