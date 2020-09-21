## Using Custom Data with an on-premise Analysis Services Live Connection

It is possible to make use of custom data with an on-premise Analysis Services live connection. This is especially pertinent to ISV's who want to utilize row level security but are unable to add every single customer to their existing AD. Custom data with an on-premise live connection is driven by the Data Gateway that provides the connection to the on-premise Analysis Services

For context,review the following two articles published by Kasper de Jong. These articles are almost the same in content, however, it is worth reviewing both of these articles since the article flagged as detailed here, requires some detailed profiling.

[Using CustomData and SSAS with Power BI Embedded -Detailed](https://www.kasperonbi.com/using-customdata-and-ssas-with-power-bi-embedded)

[Using CustomData and SSAS with Power BI Embedded](https://www.businessintelligenceinfo.com/business-intelligence/self-service-bi/using-customdata-and-ssas-with-power-bi-embedded)

Kasper wrote two very good articles around this topic and the information below  merely supports this. 

### Guidelines

1.) Create a simple tabular model to test with. It is very useful to add the following measure to the model to track how you are authenticating to Analsysis Services.

CustomData = CustomData()

2.) Test each step. 

3.) Register your Data Gateway with a service account and ensure that you can connect to the model on premise without any issues. 

4.) Map user names if applicable to authenticate to the tabular model on premise.  Ensure that the user you are authenticating with has admin permissions to the model

5.) Change the data gateway to use custom data. You do not have to add any values here. The custom data will be passed through in the code. 

6.) Change the code in the Power BI Embedded sample. This is where it differs from the attached blog posts as the sample projects have changed. 
    In the EmbedService.cs
    Replace this:
    
    var generateTokenRequestParameters = new GenerateTokenRequest(accessLevel: "view");
    var tokenResponse = await client.Reports.GenerateTokenInGroupAsync(GroupId, report.Id, generateTokenRequestParameters);
 
   with this
    
    List<EffectiveIdentity> rowLevelSecurityIdentity = new List<EffectiveIdentity>
    {
                 new EffectiveIdentity(
                    username: "morgan",
                    roles: new List<string> {"RoleA"},
                    datasets: new List<string> { report.DatasetId.ToString()})
    };

    generateTokenRequestParameters = new GenerateTokenRequest("View", null, identities: rowLevelSecurityIdentity);
    
    var tokenResponse = await client.Reports.GenerateTokenInGroupAsync(WorkspaceId, report.Id, generateTokenRequestParameters);
    
   Note: the value for username can be any value. The filter will be applied by means of the role in Analysis Services based on the custom data that is          passed through. Using AdventureWorksDW tabular model, I used the FirstName to filter on, this could also be something more relevant like Customer ID.  
   The key is to ensure you are passing username to the effectiveidentity. The value of the username is irrelevant.

7.) Create a role in Analysis Services called RoleA. Give the Role Read Permissions.
    Ensure that the account you are using to authenticate to the Analysis Services model is a member of the role
    Add the following DAX Filter (in this example, I am filtering the Customer - FirstName 
    
    =Customer[Firstname] =CUSTOMDATA()
    


