@Library('UE4_Library@main')

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
		booleanParam(defaultValue: true, description: 'Should the project be cooked?', name: 'CookProject')
		string(defaultValue: 'ThirdPersonMap', description: 'Maps we want to cook', name: 'MapsToCook')
		string(defaultValue: 'Temp', description: "Archive Foler", name: "ArchiveFolder")
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
		stage('Project Compile')
		{
			steps
			{
				script
				{
					UE4.CompileProject(params.BuildConfig as unreal.BuildConfiguration, false, params.TargetPlatform)
				}
			}
		}
		stage('Cook')
		{
			when
			{
				expression { params.CookProject == true }
			}
			steps
			{
				script
				{
					String platform = "${params.TargetPlatform}"
					if(params.TargetPlatform == "Android")
					{
						platform = "Android_Multi"
					}
					String arguments = "-fileopenlog -ddc=InstalledDerivedDataBackendGraph -unversioned -abslog=${env.WORKSPACE}/Logs -stdout -CrashForUAT -unattended -NoLogTimes  -UTF8Output"
					UE4.CookProject(platform, "", false, arguments)
				}
			}
		}
		stage("Package")
		{
			steps
			{
				script
				{
					String platform = "${params.TargetPlatform}"
					if(params.TargetPlatform == "Android")
					{
						platform = "Android_Multi -cookflavor=Multi"
					}
					UE4.PackageProject(platform, params.BuildConfig as unreal.BuildConfiguration, "", true, false, "", "-archive -archivedirectory=${env.WORKSPACE}/${params.ArchiveFolder}")
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