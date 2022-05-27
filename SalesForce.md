# SalesForce

## SalesForce 介绍

+ 有较多模块，需要付费
+ 有自动翻译功能，但自定义的变量需要手动翻译 
+ 无需安装，显示报表与图表，提升效率大
+ 安全性高
+ 提供售后支持，可以提问题

> lighting模式：现今主要采用该模式
>
> classic模式：

## SalesForce术语

+ 对象：具有特定数据类型的属性，属性与数据库表中的列字段相似
+ + 标准对象
  + 自定义对象：有==__c==(custom)作为后缀
+ 工作流：一组逻辑规则：在特定的触发节点，进行一些操作
+ 字段：对象的一个属性；库里的一个表；存在大量默认字段未显示
+ 角色：工作职位，有上下级、同级关系；在前期阶段就要与客户确定好（树形结构，上级可以浏览下级的记录）（控制用户的访问权限）
+ 简档：定义用户可以执行的任务；他们可以看到的数据；他们对数据的操作权权限(控制用户的操作权限)
+ 活动：系统标准的对象
+ 记录：存储有关特定项目的所有信息
+ 视图：记录列表筛选器
+ 页面字段前带：必填项的标识
+ 报表：数据展示
+ 仪表盘：数据展示
+ 回收站：暂存最近删除的数据，有容量的限制
+ 自定义功能：通过SalesForce的自定义平台可以快速自建模块等等
+ 云计算：云端编译与数据处理
+ 分派：将Owner为自己的记录assign给其他人员，更改“所有人”
+ 搜索：模块内精确搜索与全局搜索
+ 读取：查看信息
+ 编辑：有权限就可对数据进行编辑操作
+ 共享：允许其他用户查看或编辑您拥有的信息
+ + 共享模式：定义用户对相互之间的信息拥有的默认组织范围访问权限级别；在确定对数据的访问权限时是否使用层次结构
  + 角色层次结构：上下级的不同权限设置（上层可以看到下层）
+ else

## 对象

+ 字段
+ + 主详细关系：相当于数据库中的主键
  + 查找关系：弱一级的主键，主记录被删除子记录无影响
+ 文本区：允许输入格式化文本、链接和图片
+ 记录类型：

## 工作流规则



一条记录占用2KB



# Day2(06-21)

+ 在页面上选择”必填“，不在字段中设置为”必填“（严格，导入时易报错）

+ 面一个记录类型和简档都对应一个页面布局

+ 流操作无法取消激活，若要更改需要另存为再激活
+ Integer Stringi = integer.valueof("1111");

+　跨对象查询，一次查询多个对象，最大返回2000条数据

```APEX
String SOSL = 'FIND {' + '测试客户' + ' OR ' + 'Test' + '} IN Name Fields RETURNING Account(name,phone),Contact(name,phone)';
List<List<SObject>> searchList = search.query(SOSL);
system.debug(searchList);
```

+ 添加数据

```APEX
List<Account> acctList = new List<Account>();
acctlist.add(new Account(Name='Sales User1'));
Insert acctlist;
```

+　更新数据

```APEX
List<Account> acctList = [SELECT Id,Name FROM Account where id='0015g00000GQVPbAAP'];
for(Account acc:acctList){
    acc.Name = 'Update Sales User1';
}
Update acctList;
```

+ 数据库操作返回结果

```
List<Account> accountList = new List<Account>();
accountList.add(new Account(Name = 'Sales User 2'));
Database.SaveResult[] saveResult = Database.Insert(accountList,FALSE);
for(Database.SaveResult result : saveResult){
    if(result.isSuccess()){
        System.debug(LoggingLevel.INFO, '*** result.getId():' + result.getId());
    }else{
        System.debug(LoggingLevel.INFO, '*** result.getErrors():' + result.getErrors());
    }
}
```

+ 数据库事务保存点

```
Savepoint savePoint = Database.setSavepoint();
Database.rollback(savePoint);
```

+ 关联查询

```
List<Account> accountList = [SELECT Id
                             , name
                             , ParentId
                             , Parent.ParentId
                             , Parent.Parent.ParentId
                             , Parent.Parent.Parent.ParentId
                             , Parent.Parent.Parent.Parent.ParentId
                             FROM Account];
System.debug(LoggingLevel.INFO, '*** accountList:' + accountList);

List<OrderDetails__c> orderDetailList = [SELECT Id
                                         , Name
                                         , Quantity__c
                                         , customOrder__r.Name
                                         , customOrder__r.remarks__c
                                         , customOrder__r.account__r.Name
                                        FROM OrderDateils__c];
System.debug(LoggingLevel.INFO, '***orderDetailsList: ' + orderDetailsList);
```

+ 自定义控制器的使用

  myFirstCtrl.cls

  ```
  public class myFirstCtrl {
      public List<Account> accList {get;set;}
  
      public myFirstCtrl(ApexPages.StandardController con){
          accList = [SELECT Id,Name FROM Account LIMIT 10];
      }
  }
  ```

  MyFirstPage.page

  ```
  public class myFirstCtrl {
      public List<Account> accList {get;set;}
  
      public myFirstCtrl(ApexPages.StandardController con){
          accList = [SELECT Id,Name FROM Account LIMIT 10];
      }
  }
  ```

  ## Day3(06-22)

  

before update:修改自身

after update:更新一个其他对象，是个过程





## Day4(07-01)

#### halper.js如何调用后台方法

1.APEX方法实例申明

2.设置参数

3.设置回调函数

4.排队

#### 交互

父->子：属性

component;子->父

application：应用级别的所有的都可接收（耗资源）



1. + 子：注册，父：声明处理事件
   + 注册：compent **name** type
   + 处理 ：                **name** type action
   + application 不用在处理中一定有 **name**

# lightning

1.appliciationEvent 应用事件

+ 子组件发出，所有组件都可以接收到
+ 耗资源

2.componentEvent 组件事件

+ 只可用于子组件向父组件发出

# Trigger

### 分类

+ 以执行顺序分类有：before（保存之前），after（保存之后）
+ 以DML类型（数据操作类型）有：insert（新增）、update（更新）、delete（删除）、undelete（恢复删除）

### 属性

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

# 批处理

应用场景：大数据量的DML操作或查询操作，如查询数据量超过50000、新增、修改和删除的数据量超过10000时

### 语法

1.实现Datebase.Batchable、Database.Stateful接口

2.实现Database.Batchable接口的start方法、excute方法、finash方法

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
