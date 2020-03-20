 # 10-minute Quick start: Improve debugging experience using Dev Spaces Connect 

 

## Objective of the lab: 
Dev Spaces provides a workflow which embeds local running applications under development into a chain of requests inside a Kubernates cluster running in AKS. This allows for easier debugging and speeds up your inner loop because there is no need to build and push docker images to try our your application in the context of a cluster. 

![Workflow](/media/master.png)
 


Learn about dev spaces here: https://www.youtube.com/watch?v=brhxU_kt2HI 

 

## Prerequisites 

1. Deploy an AKS cluster, set up azure cli and configure kubectl to connect to your cluster following this guide: 
    https://docs.microsoft.com/en-us/azure/aks/kubernetes-walkthrough 

1. Install and configure Azure Dev Spaces for your cluster and connect with your local Azure Dev Spaces tooling (azds) to it using the default namespace   following this guide: https://docs.microsoft.com/en-us/azure/dev-spaces/how-to/install-dev-spaces 

1. Install the Azure Dev Spaces extension for Visual Studio Code. https://marketplace.visualstudio.com/items?itemName=azuredevspaces.azds 

1. Install Visual Studio Code, Node and NPM.

 
## Deploy the sample application 

1. The sourcecode for the sample application is located here: https://github.com/DanielMeixner/DebugContainer  

1. Deploy the sample appliction using the following yaml file 

```
kubectl apply –f https://raw.githubusercontent.com/DanielMeixner/DebugContainer/master/yamls/3svc.yaml 
```


 ## Visit the currently running application  

1. Check out the services and pods running in the default namespace of your cluster 

1. Open a browser and open the public IP for svca 

1. Navigate to http://PUBLIC-IP-OF-SERVICE-A/api/cascade 

1. The result should look like below.  

![Default result](/media/redgreenyellow.png)

 

**Hint:** Notice this visualization represents the chain of microservices which have been called: You triggered the red service (svca) which is in the backend calling the green service (svcb) which is then calling the yellow service (svc). All responses combined are used to build up the visualization.  

## Modify the source code of your current application 

 

1. As a developer you want to modify svcb. Clone the repository down to your machine using this command: 
    ```
    git clone https://github.com/DanielMeixner/DebugContainer 
    ```

1. Open the folder "DebugContainer"  in Visual Studio Code.

1. Modify the application. To keep things really simple, modify it to be purple instead of green.   The color is being taken from an environment variable called COLOR. 

1. In Visual Studio Code open up the terminal and run “SET COLOR=purple” to set the environment variable. 

    ```
    SET COLOR=purple 
    ```

1. The port of the app is configured via environment variable as well. Run this command to set it to 8081. 
    ```
    SET MYPORT=8081 
    ```

1. After that run your application locally by running node server.js. 
    ```
    node server.js
    ```

1. The application starts up locally. When you open your browser and navigate to your app on http://localhost:8081 . The app will simply display a simple purple website.  

 
## Configure AKS to redirect to your developer machine 
1. While your application is working stand alone you can’t be sure that it behaves as expected when being called from another service or when calling dependencies. What you want to have is an environment where you can do both accept incoming calls and outgoing requests without the need for any kind of mocking. 

1. **Hint:** If you are running on Windows make sure you stop and disable the branchcache service if it is running. To do so press the windows key, type msc and hit return. In the Services dialog find the entry “BranchCache”. Right-click it, select properties.  

1. Set Startup type to “disabled” and hit “apply” to avoid an auto restart of the service. 

1. Then hit “Stop” to stop the service. 

    ![Services Dialog](/media/branchcache.png)


1. Hint: For the next steps make sure in your cluster Azure Dev Spaces is already activated and in Visual Studio Code the Azure Dev Spaces extension is already installed as mentioned in the preconditions. 

1. Hit F1 in Visual Studio Code and run the command Azure Dev Spaces: Redirect an existing Kubernetes service to my machine. 

    ![Connect dialog in VSC](/media/connect.png)

1. When asked for a service to redirect choose "svcb”. 
1. When asked for mode select "replace". 
1. When asked for a port to redirect choose "8081". 


1. It will take a few seconds and a new terminal will open. This terminal is configured to work seamless with Azure Dev Spaces.  **Hint:** On Windows you might be asked to confirm the execution of azds via dialog.


1. Provide some environment variables. To configure your service B fully specify the color again and also specify information about the endpoint which shall be called by your service B.  
    ```
    SET SERVICEENDPOINTHOST=svcc 
    SET SERVICEENDPOINTPATH=/api/cascade 
    SET SERVICEENDPOINTPORT=80 
    SET MYPORT=8081 
    SET COLOR=purple 
    ```

1. As you can see the configuration is using an endpoint which is valid inside of your AKS cluster – the name “svcc” . Because we are using the Azure Dev Spaces tooling the name of this backend will resolve correctly. 

1. Get the dependencies for the demo app using npm.
    ```
    npm install
    ```
1. Start your application in this terminal running “node server.js” in the same terminal. 
    ```
    node server.js
    ```

1. The client-side Azure Dev Spaces tooling will modify your machine in a way that traffic from within this terminal will be redirected to your cluster and name resolution of Kubernetes services will also work. Azure Dev Spaces server side tooling will make sure traffic towards svcb will be forwarded to your local developer machine. 

1. Open up the browser again with the public IP from svca, (http://PUBLIC-IP-OF-SERVICE-A>/api/cascade ) You should be able to see a vizualisation like this which proves your application running locally is now part of a chain of requests in AKS. As you see on the website your choice of color impacted the result and therefore proves that the traffic has been routed from AKS to your machine and back into the cluster.   
 
    ![Modified Result](/media/redpurplegyellow.png)
 

 

1. To disconnect your local machine from the cluster hit F1 in VSC and select Azure Dev Spaces: Disconnect current session. 

 

 
 