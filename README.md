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
