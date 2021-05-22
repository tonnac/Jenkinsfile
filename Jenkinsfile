@Library('UE4_Library@main')
import static java.util.Calendar.*

def UE4 = new unreal.UE4()

def BuildConfigChoices = UE4.GetBuildConfigurationChoices()
def TargetPlatforms = UE4.GetTargetPlatformChoices()

pipeline 
{
	agent {
        node {
            label 'master'
            customWorkspace 'C:/Users/C/source/repos/SpawnSpawn'
        }
    }
    
	options 
	{ skipDefaultCheckout() }
    
	parameters
	{
		choice(
			choices: BuildConfigChoices,
			description: "Build Configuration",
			name: "BuildConfig"
			)
        choice(
            choices: TargetPlatforms,
            description: "Target Platforms",
            name: "TargetPlatform"
            )
		// booleanParam(defaultValue: true, description: 'Should the project be cooked?', name: 'CookProject')
		string(defaultValue: 'Temp', description: "Archive Folder", name: "ArchiveFolder")
	}
    
	environment 
	{
		ProjectName		= getFolderName(this)
		
		UE4 = UE4.Initialise(ProjectName, env.ENGINE_ROOT, env.WORKSPACE)
	}
	
	stages
	{
		stage('Generate Project Files')
		{
			steps
			{
				script
				{
					def formattedDate = date.format("dd/MM/yyy")
        			println formattedDate // 18/05/1988
					bat("del.bat") 
					UE4.GenerateProjectFiles()
				}
			}
		}
		stage('Editor Compile')
		{
			steps
			{
				script
				{
					UE4.CompileProject(params.BuildConfig as unreal.BuildConfiguration, true)
				}
			}
		}
		stage('Android Project Compile')
		{
			steps
			{
				script
				{
					UE4.CompileProject(params.BuildConfig as unreal.BuildConfiguration, false, params.TargetPlatform)
				}
			}
		}
		stage('Android Cook')
		{
			// when
			// {
			// 	expression { params.CookProject == true }
			// }
			steps
			{
				script
				{
					String arguments = "-fileopenlog -ddc=InstalledDerivedDataBackendGraph -unversioned -abslog=${env.WORKSPACE}/Logs -stdout -CrashForUAT -unattended -NoLogTimes  -UTF8Output"
					UE4.CookProject("Android_Multi", "", false, arguments)
				}
			}
		}
		stage("Android Package")
		{
			steps
			{
				script
				{
					UE4.PackageProject("Android -cookflavor=Multi", params.BuildConfig as unreal.BuildConfiguration, "", true, false, "", "-archive -archivedirectory=${env.WORKSPACE}/${params.ArchiveFolder}")
				}
			}
		}
		stage('Window Project Compile')
		{
			steps
			{
				script
				{
					UE4.CompileProject(params.BuildConfig as unreal.BuildConfiguration, false)
				}
			}
		}
		stage('Windows Cook')
		{
			// when
			// {
			// 	expression { params.CookProject == true }
			// }
			steps
			{
				script
				{
					String arguments = "-fileopenlog -ddc=InstalledDerivedDataBackendGraph -unversioned -abslog=${env.WORKSPACE}/Logs -stdout -CrashForUAT -unattended -NoLogTimes  -UTF8Output"
					UE4.CookProject("WindowsNoEditor", "", false, arguments)
				}
			}
		}
		stage("Windows Package")
		{
			steps
			{
				script
				{
					UE4.PackageProject("Win64", params.BuildConfig as unreal.BuildConfiguration, "", true, false, "", "-archive -archivedirectory=${env.WORKSPACE}/${params.ArchiveFolder}")
				}
			}
		}
		// stage('Build DDC') 
		// {
		// 	steps
		// 	{
		// 		script
		// 		{
		// 			// UE4.BuildDDC()
		// 		}
		// 	}
		// }
	}
}