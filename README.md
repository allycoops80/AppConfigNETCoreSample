# AWS AppConfig .NET Core Sample Console Application

## Prerequisites
- .NET Core 3.1
- AWSSDK.Core
- AWSSDK.AppConfig
- Newtonsoft.Json
- AWS AppConfig Application, Environment, and Hosted Configuration created in your AWS Account and deployed using instructions provided in [AppConfig Immersion Day Lab](https://workshop.aws-management.tools/appconfig/) (Follow instructions to [Create an Application](https://workshop.aws-management.tools/appconfig/create-application/), [Create an Environment](https://workshop.aws-management.tools/appconfig/create-environment/), [Create Configuration Profile](https://workshop.aws-management.tools/appconfig/create-configuration-profile/) and [Deployment](https://workshop.aws-management.tools/appconfig/deployment/) manually through the console).

## Overview
This sample .NET Core console application provides guidance on how to properly call the AWS AppConfig GetConfiguration API call to retrieve your configuration data.  Furthermore, it focuses on how often the GetConfiguration API should be called and the parameters that should be used.

## How to Properly Call and Handle the Response for AWS AppConfig GetConfiguration API
To properly call the AWS AppConfig GetConfiguration API, the following should occur:

1. Call GetConfiguration API with the following parameterss:
    - Application
    - Environment
    - Configuration
    - ClientId (A unique ID generated by the client to identify the client for the configuration)

2. The GetConfiguration API response will include:

    - Content: of type MemoryStream that will need to be read and converted from a Base64 to a string representation of the JSON AWS AppConfig hosted configuration data that looks like this:
        ```
        {
            "boolEnableLimitResults": false,
            "intResultLimit": 5
        }
        ```
        Note: The Content section only appears if the system finds new or updated configuration data. If the system doesn't find new or updated configuration data, then the Content section is not returned (Null).

    - ConfigurationVersion: the configuration version returned.  This value should be stored and included in subsequent calls as the ClientConfigurationVersion parameter.

3. Store the configuration data in a local cache.  

4. Subsequent calls to the GetConfiguration API should include the ClientConfigurationVersion parameter that has the value of ConfigurationVersion returned from the initial call in the response. [See documentation for more info](https://docs.aws.amazon.com/appconfig/latest/userguide/appconfig-retrieving-the-configuration.html)


## Tuning Polling Frequency
It is recommended tuning the polling frequency of your GetConfiguration API calls based on your budget, the expected frequency of your configuration deployments, and the number of targets for a configuration. 

In this sample, the polling frequency has been tuned to every 15 seconds by setting a TTL exipration in order to help avoid excess charges.  If the TTL expiration has not expired, the configuration data stored in the local cache should be used and a call to GetConfiguration API avoided.

## To Execute the Sample Locally
To run the sample locally, you must first update the appsettings.json file with your AWS profile name and region
```json
{
  "AWS": {
    "Profile": "default",
    "Region": "us-east-1"
  }
}
```

Then, use these commands:
```bash
    $ cd AppConfigNETCoreSample
    $ dotnet run
```

----

## Files  
| File                                          | Description |  
|------:|:-------------|  
| /Helpers/MemoryStreamHelper.cs                                     | Helper that converts the Content value in the GetConfigurationResponse of type MemoryStream received from GetConfiguration API call to a string containing the JSON configuration data from the hosted configuration        |  
| /Models/AppConfigData.cs                     | A model class representing the JSON configuration data from the hosted configuration       |  
| /Services/AppConfigService.cs                         | Service class that calls Getconfiguration API          |  
| /Services/IAppConfigService.cs                      | Interface for AppConfigService |  
| /Services/AppConfigDataService.cs    | Service class that calls AppConfigService to get the configuration data and serializde the JSON configuration data from the hosted configuration into our AppConfigData model |  
| /Services/IAppConfigDataService.cs                      | Interface for AppConfigDataService |  
| /appsettings.json                      | Config file with AWS profile name used to run the application and must match a profile linked to the AWS Account used to create the AppConfig Application, Environment and Configuration in the AWS Console |  
| /AppConstants.cs                          | Static class containing values for the AppConfig Application, Environment and Configuration as well as values used to serve as the local cache for ClientConfigurationVersion and AppConfigData  |  
| /Program.cs                          | Initial entry point of the appliation |
----  


## References
- [Receiving the Configuration (From AWS AppConfig User Guide)](https://docs.aws.amazon.com/appconfig/latest/userguide/appconfig-retrieving-the-configuration.html)
- [AWS AppConfig GetConfiguration API call Request Syntax](https://docs.aws.amazon.com/appconfig/2019-10-09/APIReference/API_GetConfiguration.html#API_GetConfiguration_RequestSyntax)
- [AWS AppConfig API Reference](https://docs.aws.amazon.com/appconfig/2019-10-09/APIReference/Welcome.html)
- [AWS AppConfig Workshop](https://workshop.aws-management.tools/appconfig/)
