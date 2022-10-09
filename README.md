# RESET SERVICE PRINCIPAL SECRET AND STORE IN KEY VAULT USING AZ DEVOPS

Greetings to my fellow Technology Advocates and Specialists.

In this Session, I will demonstrate __How to Reset Service Principal Secret and Store in Key Vault Using Azure DevOps.__

| __LIVE RECORDED SESSION:-__ |
| --------- |
| __LIVE DEMO__ was Recorded as part of my Presentation in __JOURNEY TO THE CLOUD 9.0__ Forum/Platform |
| Duration of My Demo = __55 Mins 42 Secs__ |
| [![IMAGE ALT TEXT HERE](https://img.youtube.com/vi/EGIOzEpOxzE/0.jpg)](https://www.youtube.com/watch?v=EGIOzEpOxzE) |


| __USE CASE:-__ |
| --------- |
| Cloud Engineer __DOES NOT__ have access to __Azure Active Directory (AAD)__ to Reset Service Principal Secret. |
| Cloud Engineer __CANNOT ELEVATE__ rights using __PIM (Privileged Identity Management)__ to Reset Service Principal Secret. |


| __AUTOMATION OBJECTIVE:-__ |
| --------- |
| Validate If the Service Principal Exists. If __No__, Pipeline will __FAIL__. |
| Validate If Resource Group Containing Key Vault Exists. If __No Resource Group Found__, Pipeline will __FAIL__. |
| Validate If Key Vault Exists inside the Specified Resource Group. If __No Key Vault Found__, Pipeline will __FAIL__. |
| If All of the above validation is __SUCCESSFUL__, Pipeline will then Reset the Service Principal Secret and Store it in the Key Vault. |


| __IMPORTANT NOTE:-__ |
| --------- |
The YAML Pipeline is tested on __WINDOWS BUILD AGENT__ Only!!!


| __REQUIREMENTS:-__ |
| --------- |

1. Azure Subscription.
2. Azure DevOps Organisation and Project.
3. Service Principal either assigned Global Administrator Privileged Identity Management (PIM) Azure AD Role or Required Microsoft Graph API Rights (__Directory.ReadWrite.All__: Read and Write Directory Data).
4. Service Principal with Required RBAC ( __Contributor__) applied on Subscription or Resource Group(s). 
5. Azure Resource Manager Service Connection in Azure DevOps.


| __HOW DOES MY CODE PLACEHOLDER LOOKS LIKE:-__ |
| --------- |
| ![Image description](https://dev-to-uploads.s3.amazonaws.com/uploads/articles/fu9vuffcn5v8xb9pqpk8.png) |


| PIPELINE CODE SNIPPET:- | 
| --------- |

| AZURE DEVOPS YAML PIPELINE (azure-pipelines-spi-reset-secret-v1.0.yml):- | 
| --------- |

```
trigger:
  none

######################
#DECLARE PARAMETERS:-
######################
parameters:
- name: SubscriptionID
  displayName: Subscription ID Details Follow Below:-
  type: string
  default: 210e66cb-55cf-424e-8daa-6cad804ab604
  values:
  - 210e66cb-55cf-424e-8daa-6cad804ab604

- name: RGNAME
  displayName: Please Provide the Resource Group Name:-
  type: object
  default: 

- name: KVNAME
  displayName: Please Provide the Keyvault Name:-
  type: object
  default: 

- name: SPINAME
  displayName: Please Provide the Service Principal Name:-
  type: object
  default:

######################
#DECLARE VARIABLES:-
######################
variables:
  ServiceConnection: amcloud-cicd-service-connection
  BuildAgent: windows-latest

#########################
# Declare Build Agents:-
#########################
pool:
  vmImage: $(BuildAgent)

###################
# Declare Stages:-
###################

stages:

- stage: RESET_SECRET_SERVICE_PRINCIPAL 
  jobs:
  - job: RESET_SECRET_SERVICE_PRINCIPAL 
    displayName: RESET SECRET SERVICE PRINCIPAL
    steps:
    - task: AzureCLI@2
      displayName: Reset SPI Secret
      inputs:
        azureSubscription: $(ServiceConnection)
        scriptType: ps
        scriptLocation: inlineScript
        inlineScript: |
          az --version
          az account set --subscription ${{ parameters.SubscriptionID }}
          az account show
          $i = az ad sp list --display-name ${{ parameters.SPINAME }} --query [].appDisplayName -o tsv
          if ($i -eq "${{ parameters.SPINAME }}") {
            $j = az group exists -n ${{ parameters.RGNAME }}
                if ($j -eq "true") {
                  $k = az keyvault list --resource-group ${{ parameters.RGNAME }} --query [].name -o tsv		
                      if ($k -eq "${{ parameters.KVNAME }}") {
                        $spiappid = az ad sp list --display-name ${{ parameters.SPINAME }} --query [].appId -o tsv
                        $spireset = az ad sp credential reset --id $spiappid --query "password" -o tsv
                        az keyvault secret set --name ${{ parameters.SPINAME }} --vault-name ${{ parameters.KVNAME }} --value $spireset
                        echo "##################################################################"
                        echo "The Reset of Service Principal ${{ parameters.SPINAME }} secret was successful. The New Secret was then Stored inside Key Vault ${{ parameters.KVNAME }} in the Resource Group ${{ parameters.RGNAME }}!!!"
                        echo "##################################################################"
                        }				
                      else {
                      echo "####################################################################################################"
                      echo "Key Vault ${{ parameters.KVNAME }} DOES NOT EXISTS in Resource Group ${{ parameters.RGNAME }}!!!"
                      echo "####################################################################################################"
                      exit 1
                          }
                }
                else {
                echo "##################################################################"
                echo "Resource Group ${{ parameters.RGNAME }} DOES NOT EXISTS!!!"
                echo "##################################################################"
                exit 1
                    }
          }
          else {
          echo "####################################################################################################################"
          echo "Service Principal ${{ parameters.SPINAME }} DOES NOT EXISTS and hence Cannot Proceed with the Reset of Secret!!!"
          echo "####################################################################################################################"
          exit 1
              }

```

__Now, let me explain each part of YAML Pipeline for better understanding.__

| __PART #1:-__ | 
| --------- |

| __BELOW FOLLOWS PIPELINE RUNTIME VARIABLES CODE SNIPPET:-__ | 
| --------- |

```
######################
#DECLARE PARAMETERS:-
######################
parameters:
- name: SubscriptionID
  displayName: Subscription ID Details Follow Below:-
  type: string
  default: 210e66cb-55cf-424e-8daa-6cad804ab604
  values:
  - 210e66cb-55cf-424e-8daa-6cad804ab604

- name: RGNAME
  displayName: Please Provide the Resource Group Name:-
  type: object
  default: 

- name: KVNAME
  displayName: Please Provide the Keyvault Name:-
  type: object
  default: 

- name: SPINAME
  displayName: Please Provide the Service Principal Name:-
  type: object
  default:

```

| __PART #2:-__ | 
| --------- |

| __BELOW FOLLOWS PIPELINE VARIABLES CODE SNIPPET:-__ | 
| --------- |

```
######################
#DECLARE VARIABLES:-
######################
variables:
  ServiceConnection: amcloud-cicd-service-connection
  BuildAgent: windows-latest

```

| __NOTE:-__ | 
| --------- |
| Please change the values of the variables accordingly. |
| The entire YAML pipeline is build using __Runtime Parameters and Variables__. No Values are Hardcoded. |


| __PART #3:-__ | 
| --------- |

| __BELOW FOLLOWS THE CONDITIONS AND LOGIC DEFINED IN THE PIPELINE (AS MENTIONED ABOVE IN THE "AUTOMATION OBJECTIVE"):-__ | 
| --------- |

```
inlineScript: |
          az --version
          az account set --subscription ${{ parameters.SubscriptionID }}
          az account show
          $i = az ad sp list --display-name ${{ parameters.SPINAME }} --query [].appDisplayName -o tsv
          if ($i -eq "${{ parameters.SPINAME }}") {
            $j = az group exists -n ${{ parameters.RGNAME }}
                if ($j -eq "true") {
                  $k = az keyvault list --resource-group ${{ parameters.RGNAME }} --query [].name -o tsv		
                      if ($k -eq "${{ parameters.KVNAME }}") {
                        $spiappid = az ad sp list --display-name ${{ parameters.SPINAME }} --query [].appId -o tsv
                        $spireset = az ad sp credential reset --id $spiappid --query "password" -o tsv
                        az keyvault secret set --name ${{ parameters.SPINAME }} --vault-name ${{ parameters.KVNAME }} --value $spireset
                        echo "##################################################################"
                        echo "The Reset of Service Principal ${{ parameters.SPINAME }} secret was successful. The New Secret was then Stored inside Key Vault ${{ parameters.KVNAME }} in the Resource Group ${{ parameters.RGNAME }}!!!"
                        echo "##################################################################"
                        }				
                      else {
                      echo "####################################################################################################"
                      echo "Key Vault ${{ parameters.KVNAME }} DOES NOT EXISTS in Resource Group ${{ parameters.RGNAME }}!!!"
                      echo "####################################################################################################"
                      exit 1
                          }
                }
                else {
                echo "##################################################################"
                echo "Resource Group ${{ parameters.RGNAME }} DOES NOT EXISTS!!!"
                echo "##################################################################"
                exit 1
                    }
          }
          else {
          echo "####################################################################################################################"
          echo "Service Principal ${{ parameters.SPINAME }} DOES NOT EXISTS and hence Cannot Proceed with the Reset of Secret!!!"
          echo "####################################################################################################################"
          exit 1
              }

```

__NOW ITS TIME TO TEST !!!...__

| __TEST CASES:-__ | 
| --------- |

| __TEST CASE #1: SERVICE PRINCIPAL NAME EXISTS, RESOURCE GROUP AND KEY VAULT EXISTS:-__ | 
| --------- |
| __DESIRED OUTPUT: PIPELINE EXECUTED SUCCESSFULLY. SERVICE PRINCIPAL SECRET GOT RESET AND STORED IN KEY VAULT.__ |
| __SERVICE PRINCIPAL IN PLACE:-__ |
| ![Image description](https://dev-to-uploads.s3.amazonaws.com/uploads/articles/t2qk0lpda14vtpbk9ja1.JPG) |
| __PIPELINE RUNTIME VARIABLES VALUE:-__ |
| ![Image description](https://dev-to-uploads.s3.amazonaws.com/uploads/articles/xve6d670sh3dhs862ar6.JPG) |
| __PIPELINE EXECUTED SUCCESSFULLY:-__ |
| ![Image description](https://dev-to-uploads.s3.amazonaws.com/uploads/articles/hbjwhp1yjimdnyoixkka.png) |
| ![Image description](https://dev-to-uploads.s3.amazonaws.com/uploads/articles/akxhd4m5uehe0ha7tjff.png) |
| __SERVICE PRINCIPAL RESET SECRET STORED IN KEY VAULT:-__ |
| ![Image description](https://dev-to-uploads.s3.amazonaws.com/uploads/articles/mw2b9hmfs7pcedhwfr4h.JPG) |
| ![Image description](https://dev-to-uploads.s3.amazonaws.com/uploads/articles/9556mpd553lmj1cm4gk9.JPG) |
| __TEST CASE #2: SERVICE PRINCIPAL DOES NOT EXISTS, RESOURCE GROUP AND KEY VAULT EXISTS:-__ |
| __DESIRED OUTPUT: PIPELINE FAILS STATING THAT THE SERVICE PRINCIPAL DOES NOT EXISTS.__ |
| __PIPELINE FAILED:-__ |
| ![Image description](https://dev-to-uploads.s3.amazonaws.com/uploads/articles/q7pbny9lpdnzbokmrbje.png) |
| ![Image description](https://dev-to-uploads.s3.amazonaws.com/uploads/articles/z8eb1ux13yqe1fjclfin.png) |
| __TEST CASE #3: SERVICE PRINCIPAL AND KEYVAULT EXISTS BUT RESOURCE GROUP DOES NOT EXISTS:-__ |
| __DESIRED OUTPUT: PIPELINE FAILS STATING THAT THE RESOURCE GROUP DOES NOT EXISTS.__ |
| __PIPELINE FAILED:-__ |
| ![Image description](https://dev-to-uploads.s3.amazonaws.com/uploads/articles/42g6qj7n4bdoeqjhwgn5.png) |
| ![Image description](https://dev-to-uploads.s3.amazonaws.com/uploads/articles/wzhwlu3um2wiw8qgoghz.png) |
| __TEST CASE #4: SERVICE PRINCIPAL AND RESOURCE GROUP EXISTS BUT KEY VAULT DOES NOT EXISTS:-__ |
| __DESIRED OUTPUT: PIPELINE FAILS STATING THAT THE KEY VAULT DOES NOT EXISTS.__ |
| __PIPELINE FAILED:-__ |
| ![Image description](https://dev-to-uploads.s3.amazonaws.com/uploads/articles/5o5bekqwxt930q0rpqht.png) |
| ![Image description](https://dev-to-uploads.s3.amazonaws.com/uploads/articles/97w1t5lxxooz29q0i3ko.png) |

| __IMPORTANT OBSERVATION:-__ | 
| --------- |
| __The Service Principal Reset Secret Entry Does Not appear in Azure Portal though available in Key Vault.__ |
| __SECRET DETAILS IN KEY VAULT AFTER RESET:-__ |
| ![Image description](https://dev-to-uploads.s3.amazonaws.com/uploads/articles/mw2b9hmfs7pcedhwfr4h.JPG) |
| __SECRET DETAILS IN AZURE PORTAL APP REGISTRATION GUI AFTER RESET:-__ |
| ![Image description](https://dev-to-uploads.s3.amazonaws.com/uploads/articles/t2qk0lpda14vtpbk9ja1.JPG) |

__Hope You Enjoyed the Session!!!__

__Stay Safe | Keep Learning | Spread Knowledge__
