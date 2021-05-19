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
		stage('Compile')
		{
			steps
			{
				script
				{
					UE4.CompileProject(params.BuildConfig as unreal.BuildConfiguration)
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
					def arguments = "";
					if(params.TargetPlatform == "Android")
					{
						arguments += "-cookflavor=ETC2"
					}
					echo arguments
					UE4.CookProject("WindowsNoEditor", "${params.MapsToCook}")
				}
			}
		}
		stage('Build DDC') 
		{
			steps
			{
				script
				{
					UE4.BuildDDC()
				}
			}
		}
	}
}