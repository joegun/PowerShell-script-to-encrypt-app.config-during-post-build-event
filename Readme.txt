Securing your app.config of your winform or WPF application automatically


I have been looking for lots of ways of how to secure the app.config of the winform or WPF applications so that you can secure your internal data when distributing your client apps to others.
You can do that through code as shown at the following link below:

http://geekswithblogs.net/afeng/archive/2006/12/10/100821.aspx
http://www.codeproject.com/Tips/598863/EncryptionplusDecryptionplusConnectionplusStringpl

I don't like the way they do it as it won't happen automatically. You have to run the code first and manually change the connection string in your app.config that you want to deploy to the clients.

So, to do that, I love to encrypt the app.config using PowerShell Window and embedded the code in the post build event in the solution property.

The PowerShell script code is shown below. You can copy and paste it with any file name.
param(
  [String] $appPath = $(throw "Application exe file path is mandatory"),
  [String] $sectionName = $(throw "Configuration section is mandatory"),
  [String] $dataProtectionProvider = "DataProtectionConfigurationProvider"
)
 
#The System.Configuration assembly must be loaded
$configurationAssembly = "System.Configuration, Version=2.0.0.0, Culture=Neutral, PublicKeyToken=b03f5f7f11d50a3a"
[void] [Reflection.Assembly]::Load($configurationAssembly)
 
Write-Host "Encrypting configuration section..."
 
$configuration = [System.Configuration.ConfigurationManager]::OpenExeConfiguration($appPath)
$section = $configuration.GetSection($sectionName)
 
if (-not $section.SectionInformation.IsProtected)
{
  $section.SectionInformation.ProtectSection($dataProtectionProvider);
  $section.SectionInformation.ForceSave = [System.Boolean]::True;
  $configuration.Save([System.Configuration.ConfigurationSaveMode]::Modified);
}
 
Write-Host "Succeeded!"

Finally, in your post build event, you can paste the following code below:
powershell "& ""{path where the powershell script code reside}\EncryptAppConfigSection.ps1""" '$(TargetPath)' 'connectionStrings'

Just remember to change the execution setting of your PowerShell permission setting according the the following link:
http://technet.microsoft.com/library/hh847748.aspx
As courtesy, I learned this technique from the following link: http://www.pvle.be/2009/03/encrypt-appconfig-section-using-powershell-as-a-post-build-event/ 