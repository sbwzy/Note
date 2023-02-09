## 小知识点

+ 用户停用的时候，如果有该用户为收件人的邮件警告，那么不能停用该用户

### Trigger

#### 分类

+ 以执行顺序分类有：before（保存之前），after（保存之后）
+ 以DML类型（数据操作类型）有：insert（新增）、update（更新）、delete（删除）、undelete（恢复删除）

#### 属性

+ isInsert：当前操作是否正在执行**添加**操作
+ isUpdate：当前操作是否正在执行**修改**操作
+ isDelete：当前操作是否正在执行**删除**操作
+ isUndelete：当前操作是否为**在回收箱中回复数据以后**操作
+ isBefore：当前操作是否为**在save以前**操作
+ isAfter：当前操作是否为**在save以后**操作
+ **不支持BeforeUndelete操作**
+ new：返回sObject的记录的最新的数据的列表
+ newMap：返回一个ID映射最新的数据列表的Map集合
+ old：返回sObject的记录修改以前的数据的列表
+ oldMap：返回一个ID映射到以前的数据列表的Map集合
+ Trigger.new：适用于执行insert和update的trigger操作时并且类型为before的时候
+ Trigger.newMap:适用于执行before update，after insert以及after update的trigger操作
+ Trigger.old以及Trigger.oldMap适用于update和delete操作

### 批处理

应用场景：大数据量的DML操作或查询操作，如查询数据量超过50000、新增、修改和删除的数据量超过10000时

#### 语法

1.实现Datebase.Batchable、Database.Stateful接口

2.实现Database.Batchable接口的start方法、excute方法、finash方法

### 搜索功能

+ 对象要设为允许搜索的
+ 能搜索记录中文本字段（也许还有其他），不能搜索公式字段

## Tips

1.向对象的列表视图添加按钮：对象详情页面选择**适用于Salesforce Classic的搜索布局**，选择**列表视图**，点击编辑后即可添加按钮



### 发邮件

#### singleEmail(系统内模板)

```java
public static void sendEmail(String ownFirstName,String conFirstName,String conLastName,String conCompanyName,String userEmail){
    System.debug('进来发邮件');
    EmailTemplate temp =  [
        SELECT Id, Name, Subject, HtmlValue, Body, BrandTemplateId
        FROM EmailTemplate
        WHERE DeveloperName = 'Existing_Dealer_Account_RSM'
        LIMIT 1
    ];
    // 邮件内容替换，依据{0},{1},{2}依次替换
    String bodyFormat = String.format(temp.Body,new List<String>{ ownFirstName , conFirstName , conLastName , conCompanyName });
    List<Messaging.SingleEmailMessage> mailList = new List<Messaging.SingleEmailMessage>();
    Messaging.SingleEmailMessage mail = new Messaging.SingleEmailMessage();
    List<String> emalList = new List<String>();
    // 收件人
    emalList.add(userEmail);     
    mail.setToAddresses(emalList);
    mail.setSenderDisplayName(null);
    mail.setTargetObjectId(UserInfo.getUserId());
    mail.setTreatTargetObjectAsRecipient(false);
    mail.setBccSender(true);
    mail.setSaveAsActivity(false);
    mail.setTemplateId(temp.Id);
    // 邮件内容，如无替换内容，可不需要
    mail.setPlainTextBody(bodyFormat);
    mailList.add(mail);
    try {
        Messaging.sendEmail(mailList);
    } catch (Exception e) {
        System.debug(e.getMessage());
    }
}
```

#### SingleEmailMessage(代码编写邮件带附件)

```java
 public static void sendEmail(Id recordId,String getemail){
     List<String> emaildetail=getemail.split(',');//多个邮件地址用','隔开
     Pagereference page = new Pagereference('/apex/Post_Bid_MeetingPDF');
     Messaging.EmailFileAttachment attachment = new Messaging.EmailFileAttachment();
     //邮件发送创建email合集
     List<Messaging.SingleEmailMessage> emails =new List<Messaging.SingleEmailMessage>();
     page.getParameters().put('id',recordId);
     Blob pdf = !Test.isRunningTest() ? page.getContentAsPDF() : Blob.valueOf('Fake content');
     attachment.setFileName('会议纪要.pdf');
     attachment.setinline(false);
     attachment.setBody(pdf);
     String url=URL.getSalesforceBaseUrl().toExternalForm();
     String href=url+'/lightning/r/Minutes_Meeting__c/'+recordId+'/view';
     //给不同的人发邮件
     for (String item : emaildetail) {
         Messaging.SingleEmailMessage emailInfo = new Messaging.SingleEmailMessage();
         emailInfo.setSubject('有关于您的招标项目的标后会议纪要，请查收！');//邮件主题
         emailInfo.setSenderDisplayName('Salesforce 系统管理员');
         emailInfo.setPlainTextBody('有关于您的招标项目的标后会议纪要已创建，请进入系统查看。' + '\n' + href);
         emailInfo.setBccAddresses(new List<String> {item});
         //emailInfo.setToAddresses(new List<String> {emailitem});

         emailInfo.setFileAttachments(new Messaging.EmailFileAttachment[] { attachment } );

         emails.add(emailInfo);
     }
     list<Messaging.SendEmailResult> emailResult =Messaging.sendEmail(emails);
     System.debug(emailResult);

 }
```

### 保存PDF为附件

```java
public static void savePDF(Id recordId){
    try {
        Pagereference page = new Pagereference('/apex/Post_Bid_MeetingPDF');
        page.getParameters().put('id',recordId);
        Blob pdf = !Test.isRunningTest() ? page.getContentAsPDF() : Blob.valueOf('Fake content');

        Attachment attach = new Attachment(Name='标后会议PDF'+Date.today().format()+'.pdf',body=pdf,ParentId=recordId);
        insert attach;
    } catch (VisualforceException e) {
        System.debug('error');
    }
}
```



### 获取SF的连接头

```java
URL.getSalesforceBaseUrl().toExternalForm()

String url=URL.getSalesforceBaseUrl().toExternalForm();
String href=url+'/lightning/r/Minutes_Meeting__c/'+recordId+'/view';
```

### JS非空检查

```javascript
// 非空检查JS方法
isNotEmpty : function(obj){
    if(typeof obj !== "undefined" && obj !== null && obj !== ""){
        return true;
    }else{
        return false;
    }
}
```

### 标准异步调用

```js
checkTPP: function(cmp){
    var action = cmp.get("c.checkTPP");
    action.setParams({"TPP": cmp.get("{!v.TPPNumber}"), "FirstName": cmp.find("FirstName").get('v.value'), "LastName": cmp.find("LastName").get('v.value'), "Email": cmp.find("Email").get('v.value')});
    action.setCallback(this, function(response) {
        var state = response.getState();
        if (state === "SUCCESS") {
            var result = response.getReturnValue();
            console.log(result);
            // 回调操作
            
        }
        else if (state === "INCOMPLETE") {
            // 代码桩
        }
        else if (state === "ERROR") {
            var errors = response.getError();
            if (errors) {
                if (errors[0] && errors[0].message) {
                    console.log("Error message: " + errors[0].message);
                }
            } else {
                console.log("Unknown error");
            }
        }
    });
    $A.enqueueAction(action);
}
```

### 测试类书写

```java
@isTest(seeAllData=true) // 可以调用系统内的数据
@isTest
public class SaleLeadImportCtrTest{

    @isTest
    private static void testMethod(){
        String str1 = '[{"客户名称*":"中国石化1","分销商代码*":3013,"电子邮件":"mail@mail.com","省份":"浙江","城市":"合肥"},{"客户名称*":"中国石化2","分销商代码*":3013,"电子邮件":"yun","省份":"浙江","城市":"衢州"},{"客户名称*":22,"分销商代码*":77,"姓名":"name3","电子邮件":"mail@mail.com","省份":"浙江","城市":"杭州"}]';

        Test.startTest();
        SaleLeadImportCtr.insertData(str1);
        Test.stopTest();
    }
}
```

### 获取RecordTypeId

```java
Id RTId = Schema.SObjectType.Account.getRecordTypeInfosByDeveloperName().get('Distributor').getRecordTypeId();
System.debug(RTId);
```

Schema.SObjectType.对象.getRecordTypeInfosByDeveloperName().get('记录类型的**DeveloperName**').getRecordTypeId();

### Schema用法

+ #### getGlobalDescribe()

```java
Map<String, Schema.SObjectType> gd = Schema.getGlobalDescribe();
```

所有对象(sObject)的names(keys) and tokens(values)

+ #### describeSObjects(sObjectTypes)

```java
Schema.DescribeSObjectResult[] Results = Schema.describeSObjects(new String[]{'Account','opportunity'});
```

This method returns a list of fields and properties of a particular sObject or array of sObjects.

+ ####  describeDataCategoryGroups(sObjectNames)

```java
public static List<Schema.DescribeDataCategoryGroupResult> describeDataCategoryGroups(List<String> sObjectNames)
```



This method is used to return a list of categorized groups linked with a particular object or specified object.

+ #### describeTabs()

```java
Schema.DescribeTabSetResult[] tabInfo = Schema.describeTabs();
```

This method is used to return information about **Custom and Standard apps** available to current users.

### 获取选项列表值

```java
// 获取选项列表值<value,label>
public static Map<String,String> getPicklistValuesMap(String sObjectName, String sFieldName){
    Map<String,String> picValues=new Map<String,String>();
    if(Schema.getGlobalDescribe().containsKey(sObjectName)){
        Map<String,Schema.SobjectField> sObjectFieldsMap=Schema.getGlobalDescribe().get(sObjectName).getDescribe().fields.getMap();
        if(sObjectFieldsMap.containsKey(sFieldName)){
            for(Schema.PicklistEntry picklistEntry : sObjectFieldsMap.get(sFieldName).getDescribe().getPicklistValues()){
                picValues.put(picklistEntry.getValue(),picklistEntry.getLabel());
            }
        }
    }
    return picValues;
}
```

```java
// 检查选项列表范围,单选
System.debug(saleLead.Province__c);
if(saleLead.Province__c != null){
    Map<String,String> ProvinceValue = getPicklistValuesMap('saleLead__c','Province__c');
    boolean BProvinceValue = ProvinceValue.containsKey(saleLead.Province__c);
    if(BProvinceValue==false){errorMsg += '省份填写不正确,';}
}
```

```java
// 多选
if(saleLead.IC__c != null){
    System.debug(saleLead.IC__c);
    Map<String,String> ICValue = getPicklistValuesMap('saleLead__c','IC__c');
    String SIC = saleLead.IC__c;
    String[] S1 = SIC.split(';');
    for(String item:S1){
        Boolean BS1 = ICValue.containsKey(item);
        if(BS1==false){errorMsg += 'IC填写不正确,';}
    }
}
```



### Toast提示

```java
//Toast提示
showTheToast : function(theTitle, theMessage, theType) {
    var toastEvent = $A.get("e.force:showToast");
    toastEvent.setParams({
        "title": theTitle,
        "message": theMessage,
        type : theType
    });
    toastEvent.fire();
}
```

### 关闭组件

```java
// 关闭lightning组件
closeModel : function (cmp) {
    let close = $A.get("e.force:closeQuickAction");
    close.fire();
}
```

### 选项列表依赖关系

```java
// 城市DependMap
Map<String,List<String>> dependMap = new Map<String,List<String>>();
List<DependentPicklistController.PicklistValue> SPC = DependentPicklistController.getPicklistValues('SaleLead__c', 'Province__c', 'City__c');
System.debug(SPC);
for(DependentPicklistController.PicklistValue item:SPC){
    List<DependentPicklistController.PicklistValue> PList= new List<DependentPicklistController.PicklistValue>();
    if(item.dependents != null){
        PList = item.dependents;
        System.debug('!!!!!'+PList);
        List<String> itemList = new List<String>();
        for(DependentPicklistController.PicklistValue item2:PList){
            if(item2.label != null && item2.label.length()>0){
                String labelList = item2.label;
                itemList.add(labelList);
            }
        }
        String key = item.label;
        dependMap.put(key, itemList);
    }
}
System.debug(dependMap);
```

```java
/**
 * @description       : 依赖关系选项列表查询
 * @author            : robert
 * @group             : dcit-hf
 * @last modified on  : 09-27-2021
 * @last modified by  : robert
**/
public with sharing class DependentPicklistController {
    @AuraEnabled //(cacheable = true)
    public static Map<String, List<PicklistValue>> getMutiPicklistValues(
      String objApiName,
      Map<String, String> dependentFieldsMap
    ) {
      System.debug(dependentFieldsMap);
      Map<String, List<PicklistValue>> returnMap = new Map<String, List<PicklistValue>>();
      for (String key : dependentFieldsMap.keySet()) {
        String key2 = dependentFieldsMap.get(key);
        //for(String key2 :dependentFieldsMap.get(key).keySet()){
        returnMap.put(key, getPicklistValues(objApiName, key2, key));
        //}
      }
      return returnMap;
    }
    @AuraEnabled //(cacheable = true)
    public static List<PicklistValue> getPicklistValues(
      String objApiName,
      String controlField,
      String dependentField
    ) {
      List<PicklistValue> pickListValues = new List<PicklistValue>();
      if (
        String.isBlank(objApiName) ||
        String.isBlank(controlField) ||
        String.isBlank(dependentField)
      ) {
        //return pickListValues;
        objApiName = 'Account';
        controlField = 'Country__c';
        dependentField = 'State__c';
        //enable the return statement and remove the above static assignment with some valid error value to update the UI
        //return;
      }
  
      //Identify the sobject type from the object name using global schema describe function
      Schema.SObjectType targetType = Schema.getGlobalDescribe().get(objApiName);
      //Create an empty object based up on the above the sobject type to get all the field names
      Schema.sObjectType objType = targetType.newSObject().getSobjectType();
      //fetch the all fields defined in the sobject
      Map<String, Schema.SObjectField> objFieldMap = objType.getDescribe()
        .fields.getMap();
      //Get the controlling and dependent picklist values from the objFieldMap
      List<Schema.PicklistEntry> controlledPLEntries = objFieldMap.get(
          controlField
        )
        .getDescribe()
        .getPicklistValues();
      List<Schema.PicklistEntry> dependentPLEntries = objFieldMap.get(
          dependentField
        )
        .getDescribe()
        .getPicklistValues();
  
      // Wrap the picklist values using custom wrapper class – PicklistValue
      for (Schema.PicklistEntry entry : controlledPLEntries) {
        PicklistValue picklistValue = new PicklistValue(
          entry.isActive(),
          entry.isDefaultValue(),
          entry.getLabel(),
          entry.getValue()
        );
        pickListValues.add(picklistValue);
      }
      //ValidFor is an indicator value for the controlling field which is base64 encrypted
      //base64 value should be convered to 6bit grouped binary and 1 indicate the controlling field in a certain order
      //Also,validFor value only be shown when it is serialized so it should be serialized then deserialized using PicklistValue wrapper class
      for (PicklistValue plVal : deserializePLEntries(dependentPLEntries)) {
        String decodedInBits = base64ToBits(plVal.validFor);
  
        for (Integer i = 0; i < decodedInBits.length(); i++) {
          // For each bit, in order: if it's a 1, add this label to the dependent list for the corresponding controlling value
          String bit = decodedInBits.mid(i, 1);
          if (bit == '1') {
            PicklistValue dependentPLValue = new PicklistValue(
              plVal.active,
              plVal.defaultValue,
              plVal.label,
              plVal.value
            );
            //Dependent picklist value is mapped to its parent controlling value through 'dependents' attribute in the PicklistValue wrapper class
            if (pickListValues.get(i).dependents == null) {
              pickListValues.get(i).dependents = new List<PicklistValue>{
                dependentPLValue
              };
            } else {
              pickListValues.get(i).dependents.add(dependentPLValue);
            }
          }
        }
      }
      System.debug(pickListValues);
      return pickListValues;
    }
  
    private static List<PicklistValue> deserializePLEntries(
      List<Schema.PicklistEntry> plEntries
    ) {
      return (List<PicklistValue>) JSON.deserialize(
        JSON.serialize(plEntries),
        List<PicklistValue>.class
      );
    }
    //Available base64 charecters
    private static final String BASE_64_CHARS =
      '' +
      'ABCDEFGHIJKLMNOPQRSTUVWXYZ' +
      'abcdefghijklmnopqrstuvwxyz' +
      '0123456789+/';
  
    // Convert decimal to binary representation (alas, Apex has no native method 🙁
    //    eg. 4 => '100', 19 => '10011', etc.
    // Method: Divide by 2 repeatedly until 0. At each step note the remainder (0 or 1).
    // These, in reverse order, are the binary.
    private static String decimalToBinary(Integer val) {
      String bits = '';
      while (val > 0) {
        Integer remainder = Math.mod(val, 2);
        val = Integer.valueOf(Math.floor(val / 2));
        bits = String.valueOf(remainder) + bits;
      }
      return bits;
    }
  
    // Convert a base64 token into a binary/bits representation
    // e.g. 'gAAA' => '100000000000000000000'
    private static String base64ToBits(String validFor) {
      if (String.isEmpty(validFor)) {
        return '';
      }
  
      String validForBits = '';
  
      for (Integer i = 0; i < validFor.length(); i++) {
        String thisChar = validFor.mid(i, 1);
        Integer val = BASE_64_CHARS.indexOf(thisChar);
        String bits = decimalToBinary(val).leftPad(6, '0');
        validForBits += bits;
      }
  
      return validForBits;
    }
  
    //Wrapper class
  
    public class PicklistValue {
      @AuraEnabled
      public Boolean active { get; set; }
      @AuraEnabled
      public Boolean defaultValue { get; set; }
      @AuraEnabled
      public String label { get; set; }
      @AuraEnabled
      public String validFor { get; set; }
      @AuraEnabled
      public String value { get; set; }
  
      @AuraEnabled
      public List<PickListValue> dependents { get; set; }
  
      public PicklistValue() {
      }
  
      public PicklistValue(
        Boolean active,
        Boolean defaultValue,
        String label,
        String validFor,
        String value
      ) {
        this.active = active;
        this.defaultValue = defaultValue;
        this.label = label;
        this.validFor = validFor;
        this.value = value;
      }
  
      public PicklistValue(
        Boolean active,
        Boolean defaultValue,
        String label,
        String value
      ) {
        this.active = active;
        this.defaultValue = defaultValue;
        this.label = label;
        this.validFor = validFor;
        this.value = value;
      }
  
      public PicklistValue(String label, String value) {
        this.label = label;
        this.value = value;
      }
    }
  }
```

```java
/**
 * @description       : 依赖关系选项列表查询测试类
 * @author            : robert
 * @group             : dcit-hf
 * @last modified on  : 09-08-2021
 * @last modified by  : robert
**/
@isTest
public class DependentPicklistControllerTest{
    @isTest
    private static void getPicklistValues_Test(){
        Test.startTest();
        DependentPicklistController.getPicklistValues('SaleLead__c', 'Province__c', 'City__c');
        Test.stopTest();
    }
}
```

### 仿原生组件样式

```html
<aura:html tag="style">
    .cuf-content {
    padding: 0 0rem !important;
    }
    .slds-p-around--medium {
    padding: 0rem !important;
    }
    .slds-modal__content{
    overflow-y:hidden !important;
    height:unset !important;
    max-height:unset !important;
    }
    .slds-modal__container {
    width : 50% !important;
    max-width : 50% !important;
    }
</aura:html>
```

### 提交审批

```java
Approval.ProcessSubmitRequest psr = new Approval.ProcessSubmitRequest();
psr.objectId = newQuo.Id;// 待审批对象Id
psr.setProcessDefinitionNameOrId('AMPRequote');// 指定批准过程（可不指定）
psr.setSubmitterId(newQuo.OwnerId);// 提交者Id
system.debug('psr' + psr);
if(!Test.isRunningTest())Approval.ProcessResult result = Approval.process(psr);
```

### lightning页面刷新

```js
$A.get('e.force:refreshView').fire();
```

### 查找字段Label（翻译）

```java
select Id,Name,toLabel(Address) FROM Account
```

### 查找所有的字段(Must Limit 200)

```java
select FIELDS(ALL) FROM InterfaceLog__c Order by CreatedDate desc limit 20
```



### 新建记录组件，阻止跳转

```js
var createRecordEvent = $A.get("e.force:createRecord");
createRecordEvent.setParams({
    "entityApiName": "Contact",
    "defaultFieldValues": {
        "AccountId": result.AccountId,
        "RMA_Contact__c": cmp.get("{!v.recordId}"),
        "Country__c": result.Country,
        "ContactState__c": result.State,
        "City__c": result.City,
        "County__c": result.County,
        "Street__c": result.Street,
        "Zip_Code__c": result.ZipCode
    },
    "recordTypeId": result.RecordTypeId,
    //阻止创建之后跳转到详细页面
    "navigationLocation": "LOOKUP",
    "panelOnDestroyCallback": function(event) {
    }
});
createRecordEvent.fire();
```

### 刷新Lighting页面，关闭Lighting组件

```js
$A.get('e.force:refreshView').fire();
$A.get("e.force:closeQuickAction").fire();
```

### Lighting记录页面重定向，

```js
var navEvt = $A.get("e.force:navigateToSObject");
navEvt.setParams({
	"recordId": cmp.get("{!v.recordId}")
	"slideDevName": "related"
});
navEvt.fire();
```

### 强制刷新页面

```js
window.location.reload(true);
```

### PDF文本区域换行

```java
public static string creatHuanhang(string sources,integer count){
    string temp='';
    //count=20;//每行显示的字数，现设为6，根据需要设置
    if(sources!=null&&sources.length()!=0 &&sources.length()>count){
        integer si=sources.length()/count;  
        for(integer i=0;i<si;i++){
            temp+=sources.substring(i*count,i*count+count)+' ';
        }
        if(sources.length()-si*count>0){
            temp+=sources.substring(si*count, sources.length())+' ';
        }
        System.debug('TEST========'+temp);
        return temp;
    }
    return sources;
}
```

### 数字显示两位小数

```html
<th valign="top" width="11%"><apex:outputText value="${0, number, ###,###,###,###,##0.00}"><apex:param value="{!itemo.TotalPrice}" /></apex:outputText></th>
```

### 操作样式布局动态调整（通过是否展示样式参数来控制）

```html
<aura:if isTrue="{!v.isAMP}">
		<aura:html tag="style">
			.slds-modal__container {
			width : 45% ;
			max-width : 100% !important;
			height : 100% !important;
			}
			.slds-modal__content{
			overflow-y:hidden !important;
			height:unset !important;
			max-height:unset !important;
			}
		</aura:html>
	</aura:if>
```

### 系统内部数据迁移

也可用于数据创建做参考

```java
// 根据已迁移的Account来查找需要迁移的Commission_And_Service_Agreement__c对象
List<Account> newAccountList= [select Id,Company_Code__c,isImport__c,oldAccount__c from Account where Company_Code__c = '1280' and isImport__c = true];
// Map<oldId,Id>
Map<String,String> accMap = new Map<String,String>();
for(Account acc:newAccountList){
    accMap.put(acc.oldAccount__c, acc.Id);
}
System.debug(accMap.size());
Set<String> oldAcoountIdSet = accMap.KeySet();
// 获取所有字段
Map<String, Schema.SObjectField> fieldMap = Schema.SObjectType.Commission_And_Service_Agreement__c.fields.getMap();
System.debug(fieldMap);
String sql = 'select ';
for(String key : fieldMap.keySet()){
    sql += key+',';
}
sql = sql.substring(0, sql.length()-1);
sql += ' From Commission_And_Service_Agreement__c WHERE Commission_And_Service_Account__c IN: oldAcoountIdSet';
List<Commission_And_Service_Agreement__c> oldObjectList = Database.query(sql);
List<Commission_And_Service_Agreement__c> newObjectList = new List<Commission_And_Service_Agreement__c>();
// 所有需要同步的对象
System.debug(oldObjectList.size());
for(Commission_And_Service_Agreement__c item:oldObjectList){
    Commission_And_Service_Agreement__c newObj = item.clone();
    newObj.oldRecord__c = item.Id;
    newObj.isImport__c = true;
    // Important!!
    newObj.Commission_And_Service_Account__c = accMap.get(item.Commission_And_Service_Account__c);
    newObjectList.add(newObj);
}
System.debug(newObjectList);

List<Database.SaveResult> ceretedResultList =  Database.insert(newObjectList,false);
Integer i =0;
Set<String> accIdSet = new Set<String>();
for(Database.SaveResult res :  ceretedResultList){
    if(!res.isSuccess()){
        System.debug('Insert Faild:this is oldRecordId: ' + newObjectList[i].oldRecord__c);
        System.debug(res.getErrors());
    }
    i++;
}


// 处理新生成对象间的查找关系
List<Commission_And_Service_Agreement__c> createdObjectList = [SELECT Id,oldRecord__c,isImport__c,afterCommission__c,BeforeCommission__c FROM Commission_And_Service_Agreement__c WHERE isImport__c = true];
// 对象间存在查找关系<oldId,newId>
Map<String,String> newObjectLookupMap = new Map<String,String>();
newObjectLookupMap.put('','');
for(Commission_And_Service_Agreement__c newObj:createdObjectList){
    newObjectLookupMap.put(newObj.oldRecord__c, newObj.Id);
}
for(Commission_And_Service_Agreement__c newObj:createdObjectList){
    newObj.afterCommission__c = newObjectLookupMap.get(newObj.afterCommission__c);
    newObj.BeforeCommission__c = newObjectLookupMap.get(newObj.BeforeCommission__c);
}
update createdObjectList;
```

### 代码调用接口（Post）

```java
// post向外服务器传输对象并获得返回
public without sharing class SFInfoCreateInterface {
    // 接口申明
	@future (callout = true)
	public static void createSFCSDelivery(String accountId) {
		createSFCSImmediately(accountId);
	}
    public static void createSFCSImmediately(String accountId) {
        Map<String, String> returnMsgs = new Map<String, String>();
        // 自定义对象 InterfaceLog__c，用于保存每次调用的结果
		InterfaceLog__c infLog = new InterfaceLog__c();
        infLog.InterfaceClassName__c = 'SFInfoCreateInterface';
        infLog.InterfaceName__c = 'SF主动推送客户信息到ESB';
        
        // 传数据内部类
        theAccountData accData = new theAccountData();
        List<Account> AccountList=[select id,SalesOrganization__c,SalesOrganizationSY__c,SYSAPSynchronizationStatus__c,JYInforSynchronizationStatus__c,name,Customer_code__c,prefix__c,paymentterm__c,language__c,CountryCode__c,City__c,fingroup__c,invmethod__c,OuZhouInforCode__c,UserDepartment__c,CurrencyIsoCode,JYcreditbalanceCurrency__c,creditbalanceCurrency__c from account where id=:accountId];
        Account updateAcc = new Account();
        if(AccountList.size()>0){
            updateAcc = AccountList[0];
        }
        try{ 
            // 需要额外处理的字段
            String accountCurrency='';
            
            if(AccountList.size()>0){
                //币种 payByBusinessPartnerCurrency
                if(updateAcc.UserDepartment__c=='家用事业部'){
                    accountCurrency=updateAcc.JYcreditbalanceCurrency__c;
                }else if(updateAcc.UserDepartment__c=='商用事业部'){
                    accountCurrency=updateAcc.creditbalanceCurrency__c;
                }else{
                    accountCurrency=updateAcc.CurrencyIsoCode;
                }
                
                System.debug(accountCurrency);

                accData.prefix = '';//updateAcc.prefix__c; // 非必填
                accData.currencys = accountCurrency;
                accData.language = '';//updateAcc.language__c; // 非必填
                accData.bpname = updateAcc.name;
                accData.country = updateAcc.CountryCode__c;
                accData.city = updateAcc.City__c;
                accData.street = '';// updateAcc.street__c;
                accData.housenr = '';// updateAcc.Company_Address__c;
                accData.postal = '';// '/';
                accData.fingroup = '';//updateAcc.fingroup__c; // 非必填
                accData.invmethod = '';//updateAcc.invmethod__c; // 非必填
                accData.paymentterm = updateAcc.paymentterm__c == null ? '/' : updateAcc.paymentterm__c; // 非必填
                accData.sfcode = '';//updateAcc.Customer_code__c; // 非必填
                accData.unit = 'Sanhua-Europe';
            }
            
            
            //传参JSON
            String bodyJson = JSON.serialize(accData);
            System.debug('bodyJson: ' + bodyJson);
            //接口body
            infLog.SyncBody__c = bodyJson;
            
            //封装请求
            Http htp = new Http();
            HttpRequest request = new HttpRequest();
            request.setBody(bodyJson);
            // 自定义标签设置请求地址：http://60.190.193.66:19088/api/Domain/ESBTest/infor/newbp?appkey=ffb9c732bdb24fa2a6cc945f7591c598
            request.setEndpoint(Label.ESBINFOR);
            request.setMethod('POST');
            request.setTimeout(120000);
            request.setHeader('Content-type', 'application/json');
            //Blob headerValue = Blob.valueOf(poApi.UserName__c + ':' + poApi.Password__c);
            //String authorizationHeader = 'Basic ' + EncodingUtil.base64Encode(headerValue);
            //request.setHeader('Authorization', authorizationHeader);
            HttpResponse res = new HttpResponse();
            if(Test.isRunningTest()){
                //测试环境
                System.debug('******in test!!');
                res.setBody('');
            }
            else{
                res = htp.send(request);
                System.debug('******not test!!');
                System.debug('*****res= '+res);
                System.debug('*****res.getBody()= '+res.getBody());
            }
            //返回数据，自定义内部类接收
            theReturnData trd = new theReturnData();

            String respStr = res.getBody();
            System.debug('原始返回：'+respStr);
            // 格式处理
            if(!Test.isRunningTest())respStr = respStr.substring(1,respStr.length()-1);
            System.debug('去除后：'+respStr);
            // 测试环境用例
            if(Test.isRunningTest())respStr ='{"status":"1"}';

            // 返回结果处理
            trd = (theReturnData)JSON.deserialize(respStr, theReturnData.class);
            System.debug(trd);
            // infLog.ErrorMsg__c=rd.msg;
            System.debug('trd.Response: ' + trd.Response);
            if(trd.Response=='Success'){
                infLog.SyncResult__c='同步成功';
                updateAcc.OuZhouInforCode__c = trd.Value;
            }else{
                infLog.SyncResult__c='同步失败';
            }
        }catch(Exception ex){
            infLog.SyncResult__c = '同步失败';
            infLog.RecordId__c = accountId;
            infLog.ErrorMsg__c = '报错行数：'+ex.getLineNumber()+':'+ex.getMessage();
            infLog.id=null;
        }

        if(updateAcc.UserDepartment__c=='家用事业部'){
            if(infLog.SyncResult__c=='同步成功'){
                updateAcc.JYInforSynchronizationStatus__c = '同步成功';
            }else if(infLog.SyncResult__c == '同步失败'){
                updateAcc.JYInforSynchronizationStatus__c = '同步失败';
            }
        }else if(updateAcc.UserDepartment__c=='商用事业部'){
            if(infLog.SyncResult__c=='同步成功'){
                updateAcc.SYInforSynchronizationStatus__c = '同步成功';
            }else if(infLog.SyncResult__c == '同步失败'){
                updateAcc.SYInforSynchronizationStatus__c = '同步失败';
            }
        }

        insert infLog;
        update updateAcc;
    }
    
    public class theAccountData {
        public String prefix;
        public String currencys;
        public String language;
        public String bpname;
        public String country;
        public String city;
        public String street;
        public String housenr;
        public String postal;
        public String fingroup;
        public String invmethod;
        public String paymentterm;
        public String sfcode;
        public String unit;
    }

    public class theReturnData {
        public String Value;
        public String Response;
    }
}
```

### JS中使用自定义标签Label

```js
$A.get("$Label.c.ApplyAccountType1")
alert('{!$Label.QuoteDeletePrice1}');
```

### APEX中使用自定义标签Label

```java
System.Label.caseCon;
```



### 获取对象类型

```java
if (atts.ParentId.getSobjectType() == Product2.SobjectType)
```

### VF中Boolean类型的判定

```html
<aura:if isTrue="{!!v.isAMPShow}"></aura:if>
<aura:if isTrue="{!v.isAMPShow}"></aura:if>
```

### 触发器实现对象下是否有文件的赋值

创建时要在ContentDocumentLink对象下

```java
public class ContentDocumentLinkTriggerHandler extends CMN_TriggerHandler {
    
    public override void onAfterInsert(List<SObject> newList, Map<Id, SObject> newMap){
    	
        Set<String> pmContentDocumentLinkSet = new Set<String>();
    	for(ContentDocumentLink clIterator : (List<ContentDocumentLink>)newList) {          
            if(clIterator.LinkedEntityId.getSobjectType() == Protocol_Management__c.SobjectType) {
                pmContentDocumentLinkSet.add(clIterator.LinkedEntityId);
            }
        }
 
        System.debug('pmContentDocumentLinkSet: '+pmContentDocumentLinkSet);
        if(!pmContentDocumentLinkSet.isEmpty()){
            List<Protocol_Management__c> updatePmList = [SELECT Id,haveDocumentation__c FROM Protocol_Management__c WHERE Id IN :pmContentDocumentLinkSet];
            if(!updatePmList.isEmpty()){
                for(Protocol_Management__c pm : updatePmList){
                    pm.haveDocumentation__c = true;
                }
                System.debug('updatePmList: '+updatePmList);
                Database.update(updatePmList, false);
            }
        }     
    }
}
```

附相关测试类逻辑

```java
ContentVersion contentVersionInsert = new ContentVersion(
            Title = 'Test',
            PathOnClient = 'Test.jpg',
            VersionData = Blob.valueOf('Test Content Data'),
            IsMajorVersion = true
        );
        insert contentVersionInsert;
 
        // Test INSERT
        ContentVersion contentVersionSelect = [SELECT Id, Title, ContentDocumentId FROM ContentVersion WHERE Id = :contentVersionInsert.Id LIMIT 1];
        List<ContentDocument> documents = [SELECT Id, Title, LatestPublishedVersionId FROM ContentDocument];
        ContentDocumentLink cDe = new ContentDocumentLink();
				cDe.ContentDocumentId = documents[0].Id;
				cDe.LinkedEntityId = am.Id; // you can use objectId,GroupId etc
				cDe.ShareType = 'I'; // Inferred permission, checkout description of ContentDocumentLink object for more details
				cDe.Visibility = 'AllUsers';
		insert cDe;
```

删除的逻辑要写在ContentDocument对象上，ContentDocumentLink上的删除触发器不触发？（有可能是isDelete赋值为true）

```java
trigger ContentDocumentTrigger on ContentDocument (before delete, after insert) {
    if(Trigger.isBefore && Trigger.isDelete){
        System.debug('进入beforeDelete');
        List<ContentDocument> oldList = Trigger.old;
        Map<Id, ContentDocument> oldMap = Trigger.oldMap;
        System.debug('oldList='+oldList);
        System.debug(oldMap.keySet());
        Set<String> pmContentDocumentSet = new Set<String>();

        List<ContentDocumentLink> cdlList = [SELECT ContentDocumentId, LinkedEntityId, Id, IsDeleted FROM ContentDocumentLink where ContentDocumentId IN :oldMap.keySet()];
        for(ContentDocumentLink clIterator : cdlList) {
            if(clIterator.LinkedEntityId.getSobjectType() == Protocol_Management__c.SobjectType) {
                pmContentDocumentSet.add(clIterator.LinkedEntityId);
            }
        }

        System.debug('pmContentDocumentSet: '+pmContentDocumentSet);
        if(!pmContentDocumentSet.isEmpty()){
            List<Protocol_Management__c> updatePmList = [SELECT Id,haveDocumentation__c,(SELECT Id, LinkedEntityId, ContentDocumentId, IsDeleted FROM ContentDocumentLinks WHERE ContentDocumentId != null) FROM Protocol_Management__c WHERE Id IN :pmContentDocumentSet];
            if(!updatePmList.isEmpty()){
                for(Protocol_Management__c pm : updatePmList){
                    if(pm.ContentDocumentLinks.size() <= 1){
                        pm.haveDocumentation__c = false;
                    }
                }
                System.debug('updatePmList: '+updatePmList);
                Database.update(updatePmList, false);
            }
        }
    }
}
```

测试类

```java
@isTest(seeAllData = true)
public with sharing class ContentDocumentTriggerTest {
    @isTest
    private static void testMethod1(){
        Protocol_Management__c pMan = [SELECT Id,haveDocumentation__c FROM Protocol_Management__c Limit 1];

        Test.startTest();
        ContentVersion cv = new ContentVersion(
            Title = 'Title_test', 
            VersionData = Blob.valueOf('Fake content'),
            PathOnClient = 'testTilePdf.pdf',
            Origin = 'C'
        );
        insert cv;

        ContentDocument cd = [
            SELECT Id
            FROM ContentDocument
            WHERE LatestPublishedVersionId = :cv.Id
            WITH SECURITY_ENFORCED
        ];

        ContentDocumentLink contentDocumentLink = new ContentDocumentLink(
            ContentDocumentId = cd.Id,
            LinkedEntityId = pMan.Id,
            ShareType = 'V'
        );
        insert contentDocumentLink;
        delete cd;
        Test.stopTest();
    }
}
```

### ContentDocument 与 ContentDocumentLink实现文件上传（带Icon）

```java
Pagereference page;
if(numMap.get(recordId) > 2){
    System.debug('page > 2');
    page = new Pagereference('/apex/AMPInvoicePage');
}else{
    System.debug('page <= 2');
    page = new Pagereference('/apex/AMPInvoicePageOne');
}
page.getParameters().put('id',recordId);
Blob pdf = !Test.isRunningTest() ? page.getContentAsPDF() : Blob.valueOf('Fake content');
System.debug('pdf:'+pdf);

// Attachment attach = new Attachment(Name = IdMap.get(recordId) + '_' + Date.today().format()+'.pdf',body=pdf,ParentId=recordId); // ai.Invoice_Number__c +'_'+ Date.today().format()
// insert attach;

ContentVersion cv = new ContentVersion(
    Title = IdMap.get(recordId) + '_' + Date.today().format(), 
    VersionData = pdf,
    PathOnClient = IdMap.get(recordId) + '_' + Date.today().format()+'.pdf',
    Origin = 'C'
);
insert cv;
System.debug(cv);

ContentDocument cd = [
    SELECT Id
    FROM ContentDocument
    WHERE LatestPublishedVersionId = :cv.Id
    WITH SECURITY_ENFORCED
];
System.debug(cd);

ContentDocumentLink contentDocumentLink = new ContentDocumentLink(
    ContentDocumentId = cd.Id,
    LinkedEntityId = recordId,
    ShareType = 'V'
);
insert contentDocumentLink;
System.debug(contentDocumentLink);
```

### 在VF页面上控制是否展示某内容（可用于PDF）

```html
<apex:outputText rendered="{!NCNR}"> <!-- NCNR为在后台类中控制的Boolean类型变量 -->
    <p style="text-align:left;font-size:11px;font-weight:bolder;">ITEMS QUOTED ARE NON-CANCELABLE NON-RETURNABLE</p>
</apex:outputText>
```

### SQL 分类并显示数量

```sql
SELECT SyncMessage,COUNT(ProductCode) AS Number FROM SAP_Product WHERE  SyncTime >= '2022-09-3 00:00' and SyncFlag = '2' group by SyncMessage
```

### 在SOQL中的WHERE条件中使用时间

```sql
select Id,createdDate,lastModifiedDate FROM Product2 WHERE lastModifiedDate > 2022-09-01T00:00:00z 
```

### 以指定字段为Key更新数据

```java
Schema.SObjectField externalIdField = ContactPonitAddressSync__c.Fields.Ship_Bill_To_ID__c;
uResults = Database.upsert (cpasList, externalIdField, false);

for (Integer i = 0; i < uResults.size(); i++){
    SObject so = upsertlist[i];
    Database.upsertResult result = uResults[i];
    System.debug(result.isSuccess());
    if (result.isSuccess()){
        so.put('Sync_Status__c', '1');
        so.put('Sync_Message__c', '1');//返回1代表upsert成功
    } else{
        String msg = '';
        for (Database.Error e : result.getErrors()){
            System.debug('Error = ' + e.getMessage());
            msg += e.getMessage();
        }
        so.put('Sync_Message__c', msg);
        so.put('Sync_Status__c', '2');
    }
}
```

### 记录上锁解锁

```java
public static void lockRecord(Set<Id> IdList){
    for(Id theId:IdList){
        if(!Approval.isLocked(theId)){//Approval.isLocked(id) 判断记录是否加锁
            Approval.lockResult ur = Approval.lock(theId);//给一条记录加锁
            if (ur.isSuccess()) {//方法执行状态
                System.debug('成功锁定记录，ID为:' + theId);
            } else {
                for(Database.Error err : ur.getErrors()) { 
                    System.debug('锁定失败'); 
                    System.debug('=============失败消息:' + err.getStatusCode() + ': ' + err.getMessage());
                }
            }
        }
    }
}


public static void unlockRecord(Set<Id> IdList){
    for(Id theId:IdList){
        if(Approval.isLocked(theId)){//Approval.isLocked(id) 判断记录是否加锁
            Approval.UnlockResult ur = Approval.unlock(theId);//给一条记录加锁
            if (ur.isSuccess()) {//方法执行状态
                System.debug('成功解锁记录，ID为:' + theId);
            } else {
                for(Database.Error err : ur.getErrors()) { 
                    System.debug('解锁失败'); 
                    System.debug('=============失败消息:' + err.getStatusCode() + ': ' + err.getMessage());
                }
            }
        }
    }
}
```

### PDF模板

```html
<!--
  @description       : 
  @last modified on  : 09-01-2022
-->
<apex:page standardController="AMP_Invoice__c" renderAs="pdf" showheader="false" standardstylesheets="false" applyBodyTag="false"
    contentType="text/html; charset=UTF-8" extensions="AMPInvoiceController">

    <head>
        <style type="text/css" media="print">
            body {
                font-family: Arial Unicode MS;
                font-size: 10px;
                font-weight: 200;
            }

            @page {
                size: A4 portrait;
                margin-left: 20px;
                margin-right: 20px;
                margin-top: 330px;
                margin-bottom: 200px;

                @top-center {
                    content: element(header);
                }

                @bottom-left {
                    content: element(footer);
                }
            }

            @page:last {
                size: A4 portrait;
                margin-left: 20px;
                margin-right: 20px;
                margin-top: 330px;
                margin-bottom: 200px;

                @top-center {
                    content: element(header);
                }

                @bottom-left {
                    content: element(footer_last);
                }
            }

            div.header {
                display: block;
                height: 330px;
                position: running(header);
            }

            div.footer {
                display: block;
                padding: 5px;
                position: running(footer);
            }

            div.footer_last {
                display: block;
                padding: 5px;
                position: running(footer);
            }

            .pagenumber:before {
                content: counter(page);
            }

            .pagecount:before {
                content: counter(pages);
            }
        </style>
    </head>

    <!-- 页眉 -->
    <div class="header" style="height:330px">
        <table style="width: 725.6px; margin-bottom:10px;">
            <tbody>
                <tr>
                    <td style="width: 176px;" rowspan="2">
                        <img src="/resource/AMPQuote" width="176" height="56" />
                    </td>
                    <td style="width: 153.6px;" rowspan="2">
                        <p style="margin: 0.6pt 0.55pt 0.0001pt 1pt; line-height: 110%; font-size: 9pt; font-family: 'Times New Roman', serif;">Applied Motion Products, Inc.</p>
                        <p style="margin: 0.6pt 0.55pt 0.0001pt 1pt; line-height: 110%; font-size: 9pt; font-family: 'Times New Roman', serif;">18645 Madrone Parkway</p>
                        <p style="margin: 0.6pt 0.55pt 0.0001pt 1pt; line-height: 110%; font-size: 9pt; font-family: 'Times New Roman', serif;">Morgan Hill CA 95037</p>
                        <p style="margin: 0cm 0cm 0cm 1pt; line-height: 10.25pt; font-size: 9pt; font-family: 'Times New Roman', serif;">408-612-4375</p>
                    </td>
                    <td style="width: 236.2px;">
                        <p style=" line-height: 30pt; font-size: 11pt; font-family: 'Times New Roman', serif;">
                            <strong>
                                <span style="font-size: 9.0pt;">&nbsp;</span>
                            </strong>
                            <span style="font-size: 9.0pt;">&nbsp;</span>
                        </p>
                    </td>
                    <td style="width: 104.6px;">
                        <p style="margin: 0.45pt 0cm 0.0001pt 1pt; font-size: 11pt; font-family: 'Times New Roman', serif; text-align: right;">
                            <span style="font-size: 16.0pt; font-family: Arial, sans-serif;">{!AMP_Invoice__c.Title__c}</span>
                        </p>
                    </td>
                </tr>
                <tr>
                    <td style="width: 236.2px;" colspan="2">
                        <div style="margin-left:70px">
                            <p style="margin: 0.6pt 0cm 0.0001pt 12.5pt; font-size: 11pt; font-family: 'Times New Roman', serif;">
                                <strong>
                                    <span style="font-size: 9.0pt;">Invoice Number:</span>
                                </strong>
                                <span style="font-size: 9pt;">{!InvoiceNumber}</span>
                            </p>
                            <p style="margin: 0.25pt 0.05pt 0.0001pt 56.25pt; text-indent: 0.45pt; line-height: 12.5pt; font-size: 11pt; font-family: 'Times New Roman', serif;">
                                <strong>
                                    <span style="font-size: 9.0pt;">Date:</span>
                                </strong>
                                <span style="font-size: 9pt;">{!Invoice_Date}</span>
                            </p>
                            <p style="margin: 0.25pt 0.05pt 0.0001pt 56.25pt; text-indent: 0.45pt; line-height: 12.5pt; font-size: 11pt; font-family: 'Times New Roman', serif;">
                                <strong>
                                    <span style="font-size: 9.0pt;">Page:</span>
                                </strong>
                                <span style="font-size: 9pt;">
                                    <span class="pagenumber" /> of
                                    <span class="pagecount" /></span>
                            </p>
                            <p style="margin: 0.25pt 0.05pt 0.0001pt 38.75pt; text-indent: 0.45pt; line-height: 12.5pt; font-size: 11pt; font-family: 'Times New Roman', serif;">
                                <strong>
                                    <span style="font-size: 9.0pt;">Currency:&nbsp;</span>
                                </strong>
                                <span style="font-size: 9.0pt;">{!AMP_Invoice__c.CurrencyIsoCode}</span>
                            </p>
                            <p style="margin: 0.25pt 0.05pt 0.0001pt 47.5pt; text-indent: 0.45pt; line-height: 12.5pt; font-size: 11pt; font-family: 'Times New Roman', serif;">
                                <strong>
                                    <span style="font-size: 9.0pt;">Ship to:</span>
                                </strong>
                                <span style="font-size: 9.0pt;">{!Ship_to_Number}</span>
                            </p>
                        </div>
                    </td>
                </tr>
            </tbody>
        </table>
        <p style="margin:  4.65pt 0cm 0.0001pt 34pt;">
            <strong>
                <span style="font-size: 9.0pt; ">Payer:</span>
            </strong>
            <span style="font-size: 9.0pt;">{!Payer}</span>
        </p>
        <table style="width: 99.9449%; height: 72.325px;">
            <tbody>
                <tr style="height: 72.325px;">
                    <td style="width: 48.0441%; height: 72.325px;">
                        <h2 style="margin: 4.65pt 0cm 0.0001pt 34pt; line-height: 9.4pt; font-size: 9pt; font-family: 'Times New Roman', serif;">BILL TO:</h2>
                        <p style="margin: 0.6pt 1.25pt 0.0001pt 34pt; line-height: 81%; font-size: 9pt; font-family: 'Times New Roman', serif;">{!BillTo1}</p>
                        <p style="margin: 0.6pt 1.25pt 0.0001pt 34pt; line-height: 81%; font-size: 9pt; font-family: 'Times New Roman', serif;">{!BillTo2}</p>
                        <p style="margin: 0 0 0 34pt; line-height: 9.4pt; font-size: 9pt; font-family: 'Times New Roman', serif;">{!BillTo3}</p>
                        <p style="margin: 0cm 0cm 0cm 34pt; line-height: 9.4pt; font-size: 9pt; font-family: 'Times New Roman', serif;">{!BillTo4}</p>
                    </td>
                    <td style="width: 48.1543%; height: 72.325px;">
                        <h2 style="margin: 4.65pt 0cm 0.0001pt 34pt; line-height: 9.4pt; font-size: 9pt; font-family: 'Times New Roman', serif;">SHIP TO:</h2>
                        <p style="margin: 0cm 0cm 0cm 34pt; line-height: 8.5pt; font-size: 9pt; font-family: 'Times New Roman', serif;">{!SHIPTo1}</p>
                        <p style="margin: 0cm 0cm 0cm 34pt; line-height: 9.4pt; font-size: 9pt; font-family: 'Times New Roman', serif;">{!SHIPTo2}</p>
                        <p style="margin: 0 0 0 34pt; line-height: 9.4pt; font-size: 9pt; font-family: 'Times New Roman', serif;">{!SHIPTo3}</p>
                        <p style="margin: 0cm 0cm 0cm 34pt; line-height: 9.4pt; font-size: 9pt; font-family: 'Times New Roman', serif;">{!SHIPTo4}</p>
                    </td>
                </tr>
            </tbody>
        </table>
        <table class="TableNormal" style="margin-left: 6.0pt; border-collapse: collapse; border: none; text-align: center;" cellspacing="0"
            cellpadding="0">
            <tbody>
                <tr style="height: 10.6pt;">
                    <td style="width: 57.5pt;border: solid black 1.0pt; background: #CCCCCC; padding: 0cm 0cm 0cm 0cm;" valign="top">
                        <p style="margin: 0cm 0cm 0cm 0cm; line-height: 9.6pt; font-size: 11pt; font-family: 'Times New Roman', serif;">
                            <strong>
                                <span style="font-size: 9.0pt; color: black;">Order</span>
                            </strong>
                        </p>
                    </td>
                    <td style="width: 60pt; border: solid black 1.0pt; border-left: none; background: #CCCCCC; padding: 0cm 0cm 0cm 0cm;" valign="top">
                        <p style="margin: 0cm 0cm 0cm 0cm; line-height: 9.6pt; font-size: 11pt; font-family: 'Times New Roman', serif;">
                            <strong>
                                <span style="font-size: 9.0pt; color: black;">Order Date</span>
                            </strong>
                        </p>
                    </td>
                    <td style="width: 80.85pt; border: solid black 1.0pt; border-left: none; background: #CCCCCC; padding: 0cm 0cm 0cm 0cm;"
                        valign="top">
                        <p style="margin: 0cm 0cm 0cm 0pt; line-height: 9.6pt; font-size: 11pt; font-family: 'Times New Roman', serif;">
                            <strong>
                                <span style="font-size: 9.0pt; color: black;">Purchase Order</span>
                            </strong>
                        </p>
                    </td>
                    <td style="width: 60.85pt; border: solid black 1.0pt; border-left: none; background: #CCCCCC; padding: 0cm 0cm 0cm 0cm;"
                        valign="top">
                        <p style="margin: 0cm 0cm 0cm 0cm; line-height: 9.6pt; font-size: 11pt; font-family: 'Times New Roman', serif;">
                            <strong>
                                <span style="font-size: 9.0pt; color: black;">Packages</span>
                            </strong>
                        </p>
                    </td>
                    <td style="width: 56.70pt; border: solid black 1.0pt; border-left: none; background: #CCCCCC; padding: 0cm 0cm 0cm 0cm;" valign="top">
                        <p style="margin: 0cm 0cm 0cm 0cm; line-height: 9.6pt; font-size: 11pt; font-family: 'Times New Roman', serif;">
                            <strong>
                                <span style="font-size: 9.0pt; color: black;">Ref. PO#</span>
                            </strong>
                        </p>
                    </td>
                    <td style="width: 72.35pt; border: solid black 1.0pt; border-left: none; background: #CCCCCC; padding: 0cm 0cm 0cm 0cm;"
                        valign="top">
                        <p style="margin: 0cm 0cm 0cm 0cm; line-height: 9.6pt; font-size: 11pt; font-family: 'Times New Roman', serif;">
                            <strong>
                                <span style="font-size: 9.0pt; color: black;">N./G.Weight</span>
                            </strong>
                        </p>
                    </td>
                    <td style="width: 82.55pt; border: solid black 1.0pt; border-left: none; background: #CCCCCC; padding: 0cm 0cm 0cm 0cm;"
                        valign="top">
                        <p style="margin: 0cm 0cm 0pt 0cm; text-align: center; line-height: 9.6pt; font-size: 11pt; font-family: 'Times New Roman', serif;">
                            <strong>
                                <span style="font-size: 9.0pt; color: black;">Ship Via</span>
                            </strong>
                        </p>
                    </td>
                    <td style="width: 100.2pt; border: solid black 1.0pt; border-left: none; background: #CCCCCC; padding: 0cm 0cm 0cm 0cm;" valign="top">
                        <p style="margin: 0cm 0cm 0cm 0cm; line-height: 9.6pt; font-size: 11pt; font-family: 'Times New Roman', serif;">
                            <strong>
                                <span style="font-size: 9.0pt; color: black;">Terms</span>
                            </strong>
                        </p>
                    </td>
                </tr>
                <tr>
                    <td>{!Order_Number}</td>
                    <td>{!Order_Date}</td>
                    <td>{!Purchase_Order}</td>
                    <td>{!Packages}</td>
                    <td>{!AMP_Invoice__c.Ref_PO__c}</td>
                    <td>{!AMP_Invoice__c.N_G_Weight__c}</td>
                    <td>{!AMP_Invoice__c.Ship_Via__c}</td>
                    <td>{!AMP_Invoice__c.Terms__c}</td>
                </tr>
            </tbody>
        </table>
        <p style="margin: 0cm; font-size: 9pt; font-family: 'Times New Roman', serif;">
            <span style="font-size: 12.0pt;">&nbsp;</span>
        </p>
    </div>
    
    <!-- 页脚 -->
    <div class="footer" style="height:200px;">
        <div style="border-top: 3px solid #000000;text-align: center;"></div>
        <div style="font-size: 12pt;">COMMENTS:</div>
    </div>

    

    <body>
        <table class="TableNormal" style="border-collapse: collapse; border: none; -fs-table-paginate: paginate; page-break-inside:avoid;" cellspacing="0" cellpadding="0">
            <thead style="display:table-header-group; text-align: center;">
                <tr style="height: 10.6pt;">
                    <th style="width: 39.7pt; border: solid black 1.0pt; background: #CCCCCC; padding: 0cm 0cm 0cm 0cm;" >
                        <p style="margin: 0cm 0cm 0cm 0cm; line-height: 9.6pt; font-size: 11pt; font-family: 'Times New Roman', serif;">
                            <strong>
                                <span style="font-size: 9.0pt; color: black;">Line/Rel</span>
                            </strong>
                        </p>
                    </th>
                    <th style="width: 113pt; border: solid black 1.0pt; border-left: none; background: #CCCCCC; padding: 0cm 0cm 0cm 0cm;">
                        <p style="margin: 0cm 0cm 0cm 0cm; line-height: 9.6pt; font-size: 11pt; font-family: 'Times New Roman', serif;">
                            <strong>
                                <span style="font-size: 9.0pt; color: black;">In Reference Order</span>
                            </strong>
                        </p>
                    </th>
                    <th style="width: 85pt; border: solid black 1.0pt; border-left: none; background: #CCCCCC; padding: 0cm 0cm 0cm 0cm;">
                        <p style="margin: 0cm 0cm 0cm 0cm; line-height: 9.6pt; font-size: 11pt; font-family: 'Times New Roman', serif;">
                            <strong>
                                <span style="font-size: 9.0pt; color: black;">Ordered</span>
                            </strong>
                        </p>
                    </th>
                    <th style="width: 85pt; border: solid black 1.0pt; border-left: none; background: #CCCCCC; padding: 0cm 0cm 0cm 0cm;">
                        <p style="margin: 0cm 0cm 0cm 0cm; line-height: 9.6pt; font-size: 11pt; font-family: 'Times New Roman', serif;">
                            <strong>
                                <span style="font-size: 9.0pt; color: black;">Shipped</span>
                            </strong>
                        </p>
                    </th>
                    <th style="width: 73.7pt; border: solid black 1.0pt; border-left: none; background: #CCCCCC; padding: 0cm 0cm 0cm 0cm;">
                        <p style="margin: 0cm 0cm 0cm 0cm; line-height: 9.6pt; font-size: 11pt; font-family: 'Times New Roman', serif;">
                            <strong>
                                <span style="font-size: 9.0pt; color: black;">Back Order</span>
                            </strong>
                        </p>
                    </th>
                    <th style="width: 85pt; border: solid black 1.0pt; border-left: none; background: #CCCCCC; padding: 0cm 0cm 0cm 0cm;">
                        <p style="margin: 0cm 0cm 0cm 0cm; line-height: 9.6pt; font-size: 11pt; font-family: 'Times New Roman', serif;">
                            <strong>
                                <span style="font-size: 9.0pt; color: black;">Unit Price</span>
                            </strong>
                        </p>
                    </th>
                    <th style="width: 99.2pt; border: solid black 1.0pt; border-left: none; background: #CCCCCC; padding: 0cm 0cm 0cm 0cm;">
                        <p style="margin: 0cm 0cm 0cm 0cm; line-height: 9.6pt; font-size: 11pt; font-family: 'Times New Roman', serif;">
                            <strong>
                                <span style="font-size: 9.0pt; color: black;">Extend Price</span>
                            </strong>
                        </p>
                    </th>
                </tr>
            </thead>

            <apex:repeat value="{!qliList}" var="itemo">
                <tbody>
                    <tr style="height:20pt;"></tr>
                    <tr style="text-align: center;">
                        <td>{!itemo.Line_Rel}</td>
                        <td>{!itemo.In_Reference_Order}</td>
                        <td>{!itemo.Ordered}</td>
                        <td>{!itemo.Shipped}</td>
                        <td>{!itemo.Back_Order}</td>
                        <td>{!itemo.Unit_Price}</td>
                        <td>{!itemo.Extend_Price}</td>
                    </tr>
                    <tr>
                        <td></td>
                        <td colspan="6" >
                            <div style="page-break-inside: avoid;">
                                <b>
                                    <div style="font-size: 9pt; font-family: 'Times New Roman', serif;">CI: {!itemo.CI}</div>
                                </b>
                                <b>
                                    <div style="font-size: 9pt; font-family: 'Times New Roman', serif;">Item: {!itemo.Item}</div>
                                </b>
                                <b>
                                    <div style="font-size: 9pt; font-family: 'Times New Roman', serif;">Ref.Item: {!itemo.Ref_Item}</div>
                                </b>
                                <b>
                                    <div style="font-size: 9pt; font-family: 'Times New Roman', serif;">Description: {!itemo.Description}</div>
                                </b>
                                <b>
                                    <div style="font-size: 9pt; font-family: 'Times New Roman', serif;">U/M: {!itemo.U_M}</div>
                                </b>
                            </div>
                        </td>
                    </tr>
                </tbody>
            </apex:repeat>
        </table>
        <p style="margin: 0.3pt 0cm 0cm; font-size: 9pt; font-family: 'Times New Roman', serif;">&nbsp;</p>
        <div style="margin:60px 100px 0px 200px">
            <table style="font-size: 9pt; font-family: 'Times New Roman', serif;">
                <tr>
                    <td>
                        <b>Packing Slip:</b>
                    </td>
                    <td>
                        <b>{!Packing_Slip}</b>
                    </td>
                </tr>
                <tr>
                    <td>
                        <b>Shipped on:</b>
                    </td>
                    <td>
                        <b>{!Shipped_on}</b>
                    </td>
                </tr>
                <tr>
                    <td>
                        <b>Tracking#:</b>
                    </td>
                    <td>
                        <b>{!AMP_Invoice__c.Tracking__c}</b>
                    </td>
                </tr>
                <tr>
                    <td>
                        <b>Service:</b>
                    </td>
                    <td>
                        <b>{!AMP_Invoice__c.Service__c}</b>
                    </td>
                </tr>
                <tr>
                    <td>
                        <b>Total Weight:</b>
                    </td>
                    <td>
                        <b>{!AMP_Invoice__c.Total_Weight__c}</b>
                    </td>
                </tr>
                <tr>
                    <td>
                        <b>Number of Packages:</b>
                    </td>
                    <td>
                        <b>{!Number_of_Packages}</b>
                    </td>
                </tr>
                <tr>
                    <td>
                        <b>End Shipment(s):</b>
                    </td>
                    <td>
                        <b>{!AMP_Invoice__c.End_Shipment_s__c}</b>
                    </td>
                </tr>
            </table>
        </div>
        <div class="footer_last" style="page-break-inside:avoid; height:200px;">
            <div style="border-top: 3px solid #000000;text-align: center;"></div>
            <table style="width: 100%; margin:10px; border-spacing: 0;">
                <tbody>
                    <tr>
                        <td style="width: 60%;" rowspan="7" valign="top">
                            <div style="font-size: 12pt;">COMMENTS:</div>
                        </td>
                        <td style="width: 20%; border: solid black; border-right: none" bgcolor="#b6b3b3" align="right">
                            <div>Sales Amount</div>
                        </td>
                        <td style="width: 20%; border: solid black; border-left: none" align="center">
                            <div>{!AMP_Invoice__c.Sales_Amount__c}</div>
                        </td>
                    </tr>
                    <tr>
                        <td style="width: 20%;" align="right">
                            <div>Misc Charges</div>
                        </td>
                        <td style="width: 20%;" align="center">
                            <div>{!AMP_Invoice__c.Misc_Charges__c}</div>
                        </td>
                    </tr>
                    <tr>
                        <td style="width: 20%;" align="right">
                            <div>Tariffs</div>
                        </td>
                        <td style="width: 20%;" align="center">
                            <div>{!AMP_Invoice__c.Tariffs__c}</div>
                        </td>
                    </tr>
                    <tr>
                        <td style="width: 20%;" align="right">
                            <div>Freight</div>
                        </td>
                        <td style="width: 20%;" align="center">
                            <div>{!AMP_Invoice__c.Freight__c}</div>
                        </td>
                    </tr>
                    <tr>
                        <td style="width: 20%;" align="right">
                            <div>Sales Tax</div>
                        </td>
                        <td style="width: 20%;" align="center">
                            <div>{!AMP_Invoice__c.Sales_Tax__c}</div>
                        </td>
                    </tr>
                    <tr>
                        <td style="width: 20%;" align="right">
                            <div>Prepaid Amount</div>
                        </td>
                        <td style="width: 20%;" align="center">
                            <div>{!AMP_Invoice__c.Prepaid_Amount__c}</div>
                        </td>
                    </tr>
                    <tr>
                        <td style="width: 20%; border: solid black; border-right: none" bgcolor="#b6b3b3" align="right">
                            <div>Total</div>
                        </td>
                        <td style="width: 20%; border: solid black; border-left: none" align="center">
                            <div>{!AMP_Invoice__c.Total__c}</div>
                        </td>
                    </tr>
                </tbody>
            </table>
        </div>
    </body>

</apex:page>
```

### 对象共享

```java
AccountShare accountShareObj = new AccountShare(AccountId = acc.Id, UserOrGroupId = acc.AgentBelongOwnerID__c, AccountAccessLevel = 'Edit', OpportunityAccessLevel = 'Edit', CaseAccessLevel = 'Edit');
```

### 屏幕流获取当前记录Id

在流中新建一个可供输入的文本变量，命名为recordId即可调用该变量以获取记录Id

### PDF table \<thead>每页重复，表格不在两页之间截断

````css
table {
    -fs-table-paginate: paginate;
    border-collapse:collapse;
    border-spacing:0px 20px; /*设置单元格间距为0*/
}
````

### VF页面中字段有值就展示

```html
<apex:outputField value="{!Fixed_Product_Request__c.AMPFixed_Product_Request__c}" rendered="{!Fixed_Product_Request__c.AMPFixed_Product_Request__c != ''}" />
```

### After Insert & update 中更新对象

```java
// 不能直接更新newList
List<ContactPointAddress> newCPAList = new List<ContactPointAddress>();
for (ContactPointAddress cpa : (List<ContactPointAddress>)newList){
    ContactPointAddress cpass = new ContactPointAddress();
    cpass.Id = cpa.Id;
    cpass.State = cpa.State;
    cpass.Region__c = cpa.Region__c;
    cpass.CountryCode = cpa.CountryCode;
    cpass.PostalCode = cpa.PostalCode;
    newCPAList.add(cpass);
}
for (ContactPointAddress cpa : newCPAList){
    // 逻辑处理
}
```

### 发送HTML格式的邮件

```java
Messaging.SingleEmailMessage email = new Messaging.SingleEmailMessage();
        email.setSenderDisplayName('the sender name you want to show');
String str='<table border="1" cellspacing="0" width = "600" height = "150"><tr><th>File Name</th><th>Original Value</th><th>New Value</th><th>Update Time</th></tr><tr><td>Row 2, Column 1</td><td>Row 2, Column 2</td><td>Row 2, Column 1</td><td>Row 2, Column 2</td></tr><tr><td>Row 2, Column 1</td><td>Row 2, Column 2</td><td>Row 2, Column 1</td><td>Row 2, Column 2</td></tr></table>';
        email.setHtmlBody(str);
        email.setSubject('test email subject use html');
        //addresses which you wanna send to
        List<String> toAddresses = new List<String>();
        toAddresses.add('1399138618@qq.com');
        email.setToAddresses(toAddresses);
        Messaging.sendEmail(new List<Messaging.SingleEmailMessage>{email});
```

### page分页与禁止此元素内分页

```html
<div style="page-break-before: auto;">
<!--
always:必定分页
auto:自动判断
...
-->

<table style="page-break-inside:avoid;" >
```









































