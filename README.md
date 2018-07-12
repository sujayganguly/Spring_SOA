# Spring_SOA
Account field"Number of Contacts", Lightning App, Integration

 1. Create a field on Account named "Number of Contacts". Populate this field with the number of contacts related to an account. 

 
 trigger CountContactOnAccount on Contact (after INSERT, after DELETE ) {
    
    Set<Id> AccountIdSet= new Set<Id>();
    
    If(Trigger.IsInsert){
        for(Contact c: Trigger.New){
            AccountIdSet.add(c.AccountId);
        }
    }
    if(Trigger.ISDelete){
        for(Contact c:Trigger.Old){
            AccountIdSet.add(c.AccountId);
        }
    }
    For(Account acc:[select id,Number_of_Contacts__c,(select id from contacts) from Account where id=:AccountIdSet]){
        System.debug('ConTact Count Show::::::'+acc.Contacts.size());
        Integer contactnumberInteger=acc.Contacts.size();
        System.debug('contactnumber'+contactnumberInteger);
        
        String Contact_NumberString=String.valueOf(contactnumberInteger);
        System.debug('contactnumberInteger'+Contact_NumberString);
        
        acc.Number_of_Contacts__c=Contact_NumberString;
        System.debug('Contact_NumberString'+Contact_NumberString);
        
        Upsert acc;
        System.debug('acc'+acc);

        
        
       // acc.Number_of_Contacts__c
        
        
    }
}





2. Build a basic lightning component that can query a list of 10 most recently created accounts and display it using a lightning app. 

Lightning App: MyAccount_APP
--------------------------------

<aura:application >
   <!-- Created By: Sujay Ganguly(Certified Salesforce developer)-->
    <c:MyContactList /> 
</aura:application>

Lightning Component: MyAccountList
--------------------------------------

<aura:component controller="MyAccountContactListController" access="global"  implements="flexipage:availableForAllPageTypes">
   
    <aura:attribute name="accounts" type="Account[]"/> 
    <aura:handler name="init" value="{!this}" action="{!c.myAction}"/>
    
<html>
<head>
    
  <!--Created By: Sujay Ganguly(Certified Salesforce developer)--> 
  
<style>
#customers {
    font-family: "Trebuchet MS", Arial, Helvetica, sans-serif;
    border-collapse: collapse;
    width: 100%;
}

#customers td, #customers th {
    border: 1px solid #ddd;
    padding: 8px;
}

#customers tr:nth-child(even){background-color: #f2f2f2;}

#customers tr:hover {background-color: #ddd;}

#customers th {
    padding-top: 12px;
    padding-bottom: 12px;
    text-align: left;
    background-color: #4CAF50;
    color: white;
}
    
caption {
  background-color: red;
  color: yellow;
  font-weight: bold;
}    
    
    .container {
  height: 100%;
  width: 100%;
  display: flex;
  position: fixed;
  align-items: center;
  justify-content: center;
}
</style>
    
</head>
<body>

<table id="customers">
   

   <caption><H1 align="center"> Account List Details </H1></caption>
   
  <tr>
    <th>Name</th>
    <th>Number of Contacts</th>
    <th>Phone</th>
    <th>Rating</th>
    <th>Type</th>
  </tr>
    
    <aura:iteration items="{!v.accounts}" var="acountList">
    
    <tr>
        <td>{!acountList.Name}</td>  
        <td>{!acountList.SujayGanG__Number_of_Contacts__c}</td> 
        <td>{!acountList.Phone}</td> 
        <td>{!acountList.Rating}</td> 
        <td>{!acountList.Type}</td> 
        
    </tr>
    </aura:iteration>
</table>
			
    
    <div class="container">
  <div><center>Most Recent Ten Account Details Are Listed Above.</center></div>
</div>
    
    
</body>
    
    
</html> 
</aura:component>

Lightning JavaScript: MyAccountListController.js
---------------------------------

({
    
    //Created By: Sujay Ganguly(Certified Salesforce developer)
	myAction : function(component, event, helper) {
		var action = component.get("c.getAccounts");
        
   			action.setCallback(this, function(data) {
			component.set("v.accounts", data.getReturnValue());

});
$A.enqueueAction(action);

	}
})

Apex: MyAccountContactListController
----------------------

public class MyAccountContactListController{

@AuraEnabled
public static List<Account> getAccounts() {

    //Created By: Sujay Ganguly(Certified Salesforce developer)

    List<Account> acountList= ([Select Id, Name,SujayGanG__Number_of_Contacts__c,Phone,BillingAddress,ShippingAddress,Rating,Type,CreatedDate From Account order by CreatedDate desc limit 10]);
    System.debug('vjjjgjjgjjjgg'+acountList);
    return acountList;
    
 }

}


3. Make a basic http callout and print the result using system.debug

@RestResource(urlMapping='/Sujay/AccountContactTest/*')
Global class RestAccountContact {

    @HttpPost  
  global static String createNewAccount(String AName, String ANumber) {
  
    //  Created By: Sujay Ganguly(Certified salesforce Developer)
    //  Workbanch API : /services/apexrest/SujayGanG/Sujay/AccountContactTest
    //  Postman API : https://sujayganguly-dev-ed.my.salesforce.com/services/apexrest/Sujay/TerritoryPostCodeTest/
    //  JSON:
    //  {   
         //   "AName":"Sujay",
         //   "ANumber":"3"
    //    } 
  
        
        List<Account> ttt= new List<Account>();
      ttt=[select id,Name,Number_of_Contacts__c from Account where Name=:AName];
      if(ttt.size()>0){
          try{
          ttt.get(0).Name=AName;
          ttt.get(0).Number_of_Contacts__c=ANumber;
         
          Update ttt;
          System.debug('Account Update Succesfully:'+ttt);
        //  Return 'Account Update Succesfully:'+ttt.get(0).id;
          }
          Catch(Exception e){
              return e.getDmlMessage(0);
          }
      }
      else{
      
          try{
      
              Account at= new Account();
              at.Name=AName;
              at.Number_of_Contacts__c=ANumber;
              
              
              Insert at;
              System.debug('Account Insert Succesfully:'+at);
             // Return 'Account Insert Succesfully:'+at.id;
              }
           catch(DMLException de)
             {
                return de.getDmlMessage(0);
             }
      
          }
        
        return null;
        
   }
}
