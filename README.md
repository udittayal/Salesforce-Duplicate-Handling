# Handling of Duplicate API Requests

Apex Trigger :

```Trigger LeadTrigger on Lead (after insert, before insert) {

    if(Trigger.isBefore && Trigger.isInsert){
        for(Lead leadObj : Trigger.new){
            if(leadObj.MobilePhone != null){
            //------- Method 1 -------------//
                            
                if(!Cache.org.contains(leadObj.MobilePhone)){
                    Cache.org.put(leadObj.MobilePhone,'',300);  //300 is minium range for lifetime of Key
                }else{
                    leadObj.addError('Idempotent Request Received');
                }
                
            //------- Method 2 -------------//
            
                //Request Id is a unique Key
                
                leadObj.Request_Id__c = leadObj.MobilePhone + String.valueOf(DateTime.now().getTime()).subString(0,10);
            }
        }
    }}
```

Python Script :

```
from concurrent.futures import ThreadPoolExecutor,as_completed
import requests

def callSFAPI():

    custom_headers = {'Authorization' : 'Bearer XXXXXXXXXXXX'}
    url = 'https://resourceful-bear-e88akt-dev-ed.trailblaze.my.salesforce.com/services/data/v58.0/sobjects/Lead/'
    payload = {"lastname" : "test", "mobilephone" : "8745818715", "company" : "self"}
    response = requests.post(url, headers=custom_headers, json=payload)
    return response.text




with ThreadPoolExecutor(max_workers=10) as executor:
    futures = [executor.submit(callSFAPI) for i in range(5)]     //5 API Calls within almost same time frame
    for future in as_completed(futures):
        print(future.result())
```


