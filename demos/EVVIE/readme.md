# E.V.V.I.E. - **E**nterprise **V**isual **V**ehicle **I**nspection **E**ngine

![EVVIE banner](https://i.imgur.com/Ca4EwdG.jpeg)

EVVIE, short for “Enterprise Visual Vehicle Inspection Engine”, is an AI-powered vehicle inspection platform that streamlines the inspection process for state and local government fleets, using artificial intelligence to analyze images of vehicles and detect damage or issues, enabling agencies to improve accuracy, reduce paperwork, and optimize maintenance management.

**Click the image below for a demonstration of E.V.V.I.E.**

[![demo video](https://i.imgur.com/5OIfuG4.png)](https://livesend.microsoft.com/i/DUFrJEz77SXgL85JbBg___Wio6___QrDyqYH7e0RigS84AIiHLC3WpVWVDDfooWSJ3PQpIq2iXxfS8jrzrqedqUQyBkGlTJR3slPbCyUqy6FOpY0dwIM38eR3YiOOSHP7___37)

*If the video above does not work, click [here](https://youtu.be/KvEFX-in2TM).*

## Architecture
EVVIE's architecture is a unique blend of Power Platform (low-code) and Azure (pro-code) resources, coming together to deliver a single unified AI-enabled service to users. The architecture is depicted below, but can be accessed in PowerPoint deck form [here](https://github.com/microsoft/SLG-Business-Applications/releases/download/35/architecture.pptx).

![architecture](https://i.imgur.com/FJ7kl5z.png)

As seen above, EVVIE uses a unique blend of Microsoft Power Platform and Microsoft Azure resources to automate the vehicle inspection process using advanced, state-of-the-art AI models. EVVIE's internal process is described below, at a high level:

A **Power Apps** (canvas) app used by the user and serves as the interface to collect photos of damage to a vehicle as part of a vehicle inspection.

The Power App leverages a **custom connector** (API integration) to provide the captured images to a custom API, an ASP.NET-based API web service.

The web service receives and parses the HTTP API call (images are encoded as a `base64` string) from the Power App, takes these photos, and interfaces with a multimodal LLM in **Azure AI Foundry** via two *agents*:
1. **Quality Validation Agent** - first, an agent specifically designed to perform a quality check of the photos is used to validate that the photos are of sufficient quality to accurately depict damage. This agent reviews the photos and ensure they are not blurry, have sufficient lighting, are within a fair distance from the vehicle (not too far away), and are not obstructed, among other things.
2. **Vehicle Damage Inspection Agent** - once the photos are determined to be of quality for the purposes of a damage assessment, a second agent, called the "Vehicle Damage Inspection Agent" is used to review the photos and classify the damage into three distrinct fields: 1) what area of car the damage is to, 2) how severe the damage is (ranked from 1-5), and 3) a general description of the damage.

The **Power Apps** inspection app receives this response back from the **ASP.NET API**, presenting the AI-created damage assessment to the user, where the user has the option of *accepting*, *modifying*, or *rejecting* the assessment altogether. After finalizing the assessment, they then submit this assessment for the given vehicle where it is securely stored in **Dataverse**.

Another **Power Apps** app (model-driven with custom pages) allows for administrators to review this vehicle inspection data.

## E.V.V.I.E. Source Code
We give EVVIE away for free! You can think of EVVIE as an accelerator: an asset that demonstrates Microsoft's technology applied to a business challenge and something you can further tweak for your *exact* business process.

You can find EVVIE's source code described below, split into two sections:

### ASP.NET-Based API Web Service
As described in the architecture diagram above, EVVIE uses a backend API running on ASP.NET to serve as a broker between the inspection app (client app) and the multimodal AI model running in Azure.

You can find the source code of EVVIE's ASP.NET-based API in the [src folder](./src/).
- [core](./src/core/) - this contains a C# console application that is used basically as a library of functions and capabilities that E.V.V.I.E. relies on for communicating with the Azure OpenAI service. This allows E.V.V.I.E. to reach out to the Azure OpenAI service to do things like identify vehicles via their license plate and assess damage to vehicles.
- [api](./src/api/) - this contains the code to a ASP.NET web service, written in C#, that exposes several endpoints that the E.V.V.I.E. interface, built in Power Apps, can call to.

You can deploy this in numerous ways, but [deploying to an Azure App Service](https://learn.microsoft.com/en-us/aspnet/core/host-and-deploy/azure-apps/?view=aspnetcore-9.0&tabs=visual-studio) is likely your best option. Be sure to modidfy the Azure OpenAI credentials to your own Azure OpenAI deployment in the [`AzureOpenAICredentialsProvider` class](./src/core/AzureOpenAICredentialsProvider.cs).

To learn more about the API, its endpoints, and their input/output schema, visit the [endpoints page](./tests/endpoints.md) where that is further discussed!

### Power Platform Solution
EVVIE's user interface and data residency is built in Microsoft's Power Platform. 

You can download the solution file (a .zip file) [here](https://github.com/microsoft/SLG-Business-Applications/releases/download/18/EVVIE_1_0_0_3.zip). This solution contains the underlying tables and option sets that make up the data structure, the Power Apps Canvas App that staff use while inspecting their vehicle, the Power Apps Model-Driven App that administrative staff can use to review these inspections, and the custom connector that is used to communicate with the EVVIE backend API system.

**After importing the solution file into your environment, be sure to update the custom connector's actions so they point to your specific Azure App Service endpoints**.

## How to Deploy EVVIE
Deploying EVVIE is not a complicated process. In its current state, EVVIE is a functioning proof of concept and can certainly provide value to an organization. Follow the steps below to deploy EVVIE:
1. **Create your own Azure OpenAI instance** - A multi-modal large language model serves as the "brain" of EVVIE. This is the AI model that reviews the photos of damage to vehicles, assess the damages, and then documents its assessment.
    1. You can follow [this guide](https://learn.microsoft.com/en-us/azure/ai-services/openai/how-to/create-resource?pivots=web-portal) on how to deploy your own Azure OpenAI model in either Azure Commercial or Azure for Government (whichever you prefer or are already using!).
    2. When you go to deploy a model, we recommend you use **GPT-4o**; As of the time of this writing, GPT-4o provides the right balance of intelligence, multi-modal capabilities, and cost savings. GPT-4o was used in the demo.
2. **Deploy the ASP.NET API** - A lightweight API system (application programming interface) serves as the intermediary between the Power App and the Azure OpenAI model.
    1. Clone (copy to your local PC) the code for this ASP.NET project in the [src folder](./src/).
    2. You want the API to know what Azure OpenAI model it should be calling to for the AI-driven assessment! You want it to be calling to the Azure OpenAI model you stood up in the last step! Grab your secret credentials (endpoint, API key, etc.) from step 1 and place them in their appropriate variables in the [`AzureOpenAICredentialsProvider` class](./src/core/AzureOpenAICredentialsProvider.cs).
    3. Follow the instructions documented [here](https://learn.microsoft.com/en-us/aspnet/core/tutorials/publish-to-azure-webapp-using-vs?view=aspnetcore-9.0) to deploy the ASP.NET project to the cloud.
3. **Deploy the Power Platform EVVIE Solution** - The Power Platform solution, found above, contains the Power Platform source code for the EVVIE interface, tables, schema, custom connector, and more.
    1. In Power Apps, go to "solutions". Click "import solution" and then select the .zip solution file you downloaded (see above).
    2. Let it import! (should take 3-5 minutes)
    3. After importing, open your solution. Open (edit) the EVVIE custom connector. Update the specific HTTP endpoints that EVVIE is set to call from *the sample* endpoints to *the actual endpoints of your ASP.NET web API*.
    4. Open, save, and publish the "EVVIE - Vehicle Inspector" canvas app.
    5. Share the app with whoever you wish to inspect vehicles. 
    6. You're done! Now you should see all inspections appear in the "Fleet Inspection Management" model-driven app.

## Marketing Material
Below you can find various forms of marketing material for EVVIE:

### One Pager
A "one-pager" flyer exists for EVVIE:

![one-pager](https://i.imgur.com/Y3dUY0M.jpeg)

The one-pager can be downloaded in various formats below:
- [As a PowerPoint deck](./one-pager/one-pager.pptx)
- [In PDF Form](./one-pager/one-pager.pdf)

### Single Slide
A one-slide description of EVVIE can be downloaded [here](https://github.com/microsoft/SLG-Business-Applications/releases/download/25/one-pager-horizontal.pptx). See the preview below:

![evvie](https://i.imgur.com/kdquuAR.jpeg)

## Other Supplied Resources
- [EVVIE's changelog](./changelog.md), documenting every version of EVVIE.
- Find and use EVVIE's graphics [here](./graphics/).
- Read more about EVVIE's ASP.NET-based API, endpoints, and input/output schema [here](./tests/endpoints.md).
- Get sample imagery, already encoded in base64, to test with [here](./tests/sample-images/).

## Development Team
EVVIE was created by the U.S. State & Local Government team at Microsoft:
- [Tim Hanewich](https://www.linkedin.com/in/timhanewich), *Power Platform Technical Specialist* - Complete Design and Implementation
- [Kelly Cason](https://www.linkedin.com/in/kellycason/), *Business Applications Technical Specialist* - UI & UX Design