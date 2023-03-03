#  Ingress Nginx Auth with Azure Active Directory via oauth2

![oauth2-topology](https://user-images.githubusercontent.com/95440965/222792616-4eaef278-baa9-44e9-af71-ae7e77f1cd51.jpg)

<h3>In this repository, you will learn how to deploy oauth2-proxy on Kubernetes using the Azure Active Directory Provider. This setup can be used in conjunction with the Ingress Nginx Controller, which allows us to protect our applications. Additionally, we will use Redis to store session cache. <h3>

  
# Registering your application
<h3>  
First step, register your application on Azure Active Directory

To add an application: go to https://portal.azure.com, choose Azure Active Directory, select App registrations and then click on New registration.
Pick a name, check the supported account type(single-tenant, multi-tenant, etc):<h3>

![1cria app registration](https://user-images.githubusercontent.com/95440965/222792489-6ada6d42-a7b9-4ecb-b903-9dec2c580a41.gif)
<h3>
In the Redirect URI section create a new Web platform entry for each app that you want to protect by the oauth2 proxy(example: https://example.domain.com.br/oauth2/callback).Click Register.<h3>

![2cria urls redirects](https://user-images.githubusercontent.com/95440965/222792517-3032fb33-389f-4943-86c3-2fdddfa40dcc.gif)
<h3>
Next we need to add group read permissions for the app registration, on the API Permissions page of the app, click on Add a permission, select Microsoft Graph, then select Application permissions, then click on Group and select Group.Read.All. Hit Add permissions and then on Grant admin consent (you might need an admin to do this).<h3>


![3 permissoes](https://user-images.githubusercontent.com/95440965/222809833-dd56efca-1d1c-4efc-b5de-69a2fe9fb528.gif)

  
  # Generating Cookie Secret
<h3>
Then, we need to generate a strong cookie secret, you can generate using one of the options bellow:
<h3>
<h2>Powershell:

Add-Type -AssemblyName System.Web
[Convert]::ToBase64String([System.Text.Encoding]::UTF8.GetBytes([System.Web.Security.Membership]::GeneratePassword(32,4))).Replace("+","-").Replace("/","_")
<h2> 
<h2>Bash:

dd if=/dev/urandom bs=32 count=1 2>/dev/null | base64 | tr -d -- '\n' | tr -- '+/' '-_'; echo
<h2> 
<h2>Python:

python -c 'import os,base64; print(base64.urlsafe_b64encode(os.urandom(32)).decode())'
<h2> 
  
 
# Running Github Worflow
<h3>
After that you will have the infos we need: 

Tenant ID, Client ID, , Client Secret, Cookie Secret and Domain.


With this information, you can trigger the workflow on the repository, and it will generate the YAML file for deploying oauth2-proxy.

![workflow](https://user-images.githubusercontent.com/95440965/222804440-fc1144c5-5e00-45b9-b204-c88ed2ea5d4f.gif)

The deployment process involves a single YAML file containing two deployments and two services for oauth2-proxy and redis. It is important to note that these deployments and services should be located in the same namespace as the applications and ingress.


1.	Deployment (Docker image: oauth2-proxy:latest, with the provider configuration for azure active directory)
2.	Service (Service running on tcp port 4180)
3.	Deployment (Docker image: redis:alpine, single pod redis deployment )
4.	Service (Service running on tcp port 6379).


Then you run the command "kubectl apply -f oauth2-proxy.yml' on the yaml file generated with the tenand and client info.

![image](https://user-images.githubusercontent.com/95440965/222804591-9c2e3e3d-4c65-4bb4-96f3-f0fd8f268595.png)

After the deploy you can check de pod logs and it will be like this:<h3>

![logs-pod-oauth2](https://user-images.githubusercontent.com/95440965/222797191-6fe5f79e-f397-424b-94d7-5361b0e664bb.jpg)

<h3>
And it's working! To use it with your ingress, you need to create two Ingress objects: one for the backend service (with two annotations for authorization with Nginx), and the other for the authentication service (using oauth2-proxy). Please refer to the example file, oauth2-ingress.yml, for guidance on how to create these objects in Kubernetes.<h3>

![funcionando](https://user-images.githubusercontent.com/95440965/222796708-20061975-6670-49e6-9d3a-91595043de86.gif)



