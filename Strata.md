@isTest
public with sharing class TestStrataAPI {

   public static testMethod void testProcessProductSuccess()
   { 
      Account acct = TestObjectFactory.CreateAccount('test');
      insert acct;
      Product_Type_Category__c category = new Product_Type_Category__c(Name='Storage');
      insert category;
      Asset asst = TestObjectFactory.CreateAsset('test', acct.id);
      asst.Product_Type_Category__c = category.id;
      asst.text_product_description__c = 'bold descritpion';
      asst.webpage__c = 'https://www.example.com';
      asst.brief_Overview__c = 'a brief overview';
      insert asst;
      Partner_Product_Versions__c aVersion = new Partner_Product_Versions__c();
      aVersion.version__c = '1';
      aVersion.availability__c = true;
      aVersion.catalog_visibility__c = false;
      aVersion.partner_product__c = asst.id;
      insert aVersion;
      Catalog_Configuration__c config = TestObjectFactory.CreateCatalogConfig();
      insert config;
      Test.setMock(HttpCalloutMock.class, new StrataApiMock(StrataApiMock.method.CREATE_PRODUCT, StrataApiMock.status.SUCCESS));
      Test.startTest();
      string result = StrataAPIAccess.processProduct(asst.id, '159545');
      Asset anAsset = [Select catalog_product_id__c, Link_to_UCC__c,catalog_sync_status__c,catalog_sync_message__c,catalog_Status_Code__c from Asset where id = : asst.id limit 1];
      System.AssertEquals('https://access.stage.redhat.com/ecosystem/software/2285473', anAsset.Link_to_UCC__c, 'Create Product view uri should have been returned');
      System.AssertEquals('2285473', anAsset.catalog_product_id__c, 'Product id 2285473 should have been returned');
      System.AssertEquals('Created', anAsset.catalog_sync_message__c, 'Status message should have been created');
      System.AssertEquals(200, anAsset.catalog_Status_Code__c,'The status code should have been 200');
      System.AssertEquals('Success', result, 'Result should have been success');
      Test.stopTest();
   }

   public static testMethod void testCreateProductError()
   {

      Account acct = TestObjectFactory.CreateAccount('acct1');
      insert acct;
      Product_Type_Category__c category = new Product_Type_Category__c(Name='Storage');
      insert category;
      Asset asst = TestObjectFactory.CreateAsset('test', acct.id);
      asst.Product_Type_Category__c = category.id;
      asst.webpage__c = 'https://www.example.com';
      insert asst;
      Catalog_Configuration__c config = TestObjectFactory.CreateCatalogConfig();
      insert config;
      Test.setMock(HttpCalloutMock.class, new StrataApiMock(StrataApiMock.method.CREATE_PRODUCT, StrataApiMock.status.FAILURE));
      Test.startTest();
      string result = StrataAPIAccess.ProcessProduct(asst.id, '1591591');
      System.AssertEquals('Error', result,'result should have been Error');
      asst = [Select catalog_sync_status__c,catalog_sync_message__c,catalog_Status_Code__c  from Asset where id = :asst.id Limit 1];
      System.AssertEquals('Service Unavailable', asst.catalog_sync_message__c, 'Status message should have been Service Unavailable');
      System.AssertEquals(503, asst.catalog_Status_Code__c,'The status code should have been 503');
      Test.stopTest();
   }

   public static testMethod void testUpdateProductSuccess()
   {
      Account acct = TestObjectFactory.CreateAccount('test');
      insert acct;
      Product_Type_Category__c category = new Product_Type_Category__c(Name='Storage');
      insert category;
      Asset asst = TestObjectFactory.CreateAsset('test', acct.id);
      asst.Product_Type_Category__c = category.id;
      asst.webpage__c = 'https://www.webpage.com';
      asst.catalog_product_id__c = '22222222';
      insert asst;
      Catalog_Configuration__c config = TestObjectFactory.CreateCatalogConfig();
      insert config;
      Test.startTest();
      Test.setMock(HttpCalloutMock.class, new StrataApiMock(StrataApiMock.method.UPDATE_PRODUCT, StrataApiMock.status.SUCCESS));
      string result = StrataAPIAccess.ProcessProduct(asst.id,'121212');
      Asset anAsset = [Select catalog_product_id__c, Link_to_UCC__c,catalog_sync_status__c,catalog_sync_message__c,catalog_Status_Code__c from Asset where id = : asst.id limit 1];
      System.AssertEquals('https://access.qa.redhat.com/ecosystem/software/2461673', anAsset.Link_to_UCC__c, 'Create Product view uri should have been returned');
      System.AssertEquals('2461673', anAsset.catalog_product_id__c, 'Product id 2461673 should have been returned');
      System.AssertEquals('OK', anAsset.catalog_sync_message__c, 'Status message should have been OK');
      System.AssertEquals(200, anAsset.catalog_Status_Code__c,'The status code should have been 200');
      System.AssertEquals('Success', result, 'Result should have been success');
      Test.stopTest();
   }

   public static testMethod void testUpdateProductError()
   {
      Account acct = TestObjectFactory.CreateAccount('acct1');
      insert acct;
      Product_Type_Category__c category = new Product_Type_Category__c(Name='Storage');
      insert category;
      Asset asst = TestObjectFactory.CreateAsset('test', acct.id);
      asst.Product_Type_Category__c = category.id;
      asst.webpage__c = 'https://www.example.com';
      insert asst;
      Catalog_Configuration__c config = TestObjectFactory.CreateCatalogConfig();
      insert config;
      Test.startTest();
      Test.setMock(HttpCalloutMock.class, new StrataApiMock(StrataApiMock.method.UPDATE_PRODUCT, StrataApiMock.status.FAILURE));
      string result = StrataAPIAccess.ProcessProduct(asst.id,'123123');
      System.AssertEquals('Error', result,'result should have been error');
      //System.AssertEquals('A product must be sent to the catalog before it can be updated', result,'result should have been error');
      asst = [Select catalog_sync_status__c,catalog_sync_message__c,catalog_Status_Code__c  from Asset where id = :asst.id Limit 1];
      //System.AssertEquals('Service Unavailable', asst.catalog_sync_message__c, 'Status message should have been Service Unavailable');
      //System.AssertEquals(503, asst.catalog_Status_Code__c,'The status code should have been 503');
   }

   public static testMethod void testCreateCertificationSuccess()
   {
      RhZone__c zone = TestObjectFactory.CreateZone('Middleware');
      insert zone;
      Project__c project = TestObjectFactory.CreateProject('testProject');
      project.zone__c = zone.id;
      insert project;
      Catalog_Configuration__c config = TestObjectFactory.CreateCatalogConfig();
      insert config;
      Test.StartTest();
      Test.setMock(HttpCalloutMock.class, new StrataApiMock(StrataApiMock.method.CREATE_CERTIFICATION, StrataApiMock.status.SUCCESS));
      string result = StrataAPIAccess.processCertification(project.id);

      System.AssertEquals('Success', result,'result should have been success');
      project = [Select catalog_certification_id__c, catalog_URL__c, catalog_sync_status__c, catalog_sync_message__c, catalog_Status_Code__c from Project__c where id = :project.id limit 1];
      System.AssertEquals('https://access.stage.redhat.com/content/2285483', project.catalog_URL__c, 'Create Catalog view uri should have been returned');
      System.AssertEquals('2285483', project.catalog_certification_id__c, 'Catalog certification id 2285483 should have been returned');
      System.AssertEquals('Created', project.catalog_sync_message__c, 'Status message should have been created');
      System.AssertEquals(200, project.catalog_Status_Code__c,'The status code should have been 200');
      Test.stopTest();
   }

   public static testMethod void testCreateCertificationError()
   {
      RhZone__c zone = TestObjectFactory.CreateZone('Middleware');
      insert zone;
      Project__c project = TestObjectFactory.CreateProject('Project23');
      project.zone__c = zone.id;
      insert project;
      Catalog_Configuration__c config = TestObjectFactory.CreateCatalogConfig();
      insert config;
      Test.startTest();
      Test.setMock(HttpCalloutMock.class, new StrataApiMock(StrataApiMock.method.CREATE_CERTIFICATION, StrataApiMock.status.FAILURE));
      string result = StrataAPIAccess.processCertification(project.id);
      System.AssertEquals('Error', result,'result should have been Error - send to catalog');
      project = [Select catalog_URL__c, catalog_sync_status__c, catalog_sync_message__c, catalog_Status_Code__c from Project__c where id = :project.id limit 1];
      System.AssertEquals('Service Unavailable', project.catalog_sync_message__c, 'Status message should have been Service Unavailable');
      System.AssertEquals(503, project.catalog_Status_Code__c,'The status code should have been 503');
      Test.stopTest();
   }

   public static testMethod void testUpdateCertificationSuccess()
   {
      RhZone__c zone = TestObjectFactory.CreateZone('OpenStack');
      insert zone;
      Project__c project = TestObjectFactory.CreateProject('Project23');
      project.zone__c = zone.id;
      project.catalog_certification_id__c = '2323232';
      insert project;
      Catalog_Configuration__c config = TestObjectFactory.CreateCatalogConfig();
      insert config;
      Test.startTest();
      Test.setMock(HttpCalloutMock.class, new StrataApiMock(StrataApiMock.method.UPDATE_CERTIFICATION, StrataApiMock.status.SUCCESS));
      string result = StrataAPIAccess.processCertification(project.id);

      System.AssertEquals('Success', result,'result should have been success');
      project = [Select catalog_certification_id__c, catalog_URL__c, catalog_sync_status__c, catalog_sync_message__c, catalog_Status_Code__c from Project__c where id = :project.id limit 1];
      System.AssertEquals('https://access.qa.redhat.com/content/2368123', project.catalog_URL__c, 'Catalog view uri should have been returned');
      System.AssertEquals('2368123', project.catalog_certification_id__c, 'Catalog certification id 2368123 should have been returned');
      System.AssertEquals('OK', project.catalog_sync_message__c, 'Status message should have been created');
      System.AssertEquals(200, project.catalog_Status_Code__c,'The status code should have been 200');
      Test.stopTest();
   }

   public static testMethod void testUpdateCertificationError()
   {
      RhZone__c zone = TestObjectFactory.CreateZone('OpenStack');
      insert zone;
      Project__c project = TestObjectFactory.CreateProject('Project23');
      project.catalog_certification_id__c = '2323232';
      insert project;
      Catalog_Configuration__c config = TestObjectFactory.CreateCatalogConfig();
      insert config;
      Test.startTest();
      Test.setMock(HttpCalloutMock.class, new StrataApiMock(StrataApiMock.method.UPDATE_CERTIFICATION, StrataApiMock.status.FAILURE));
      string result = StrataAPIAccess.processCertification(project.id);
      System.AssertEquals('Error', result,'result should have been Error - update catalog');
      project = [Select catalog_URL__c, catalog_sync_status__c, catalog_sync_message__c, catalog_Status_Code__c from Project__c where id = :project.id limit 1];
      System.AssertEquals('Service Unavailable', project.catalog_sync_message__c, 'Status message should have been Service Unavailable');
      System.AssertEquals(503, project.catalog_Status_Code__c,'The status code should have been 503');
      Test.stopTest();
   }
   public static testMethod void testPublish()
   {
      RhZone__c zone = TestObjectFactory.CreateZone('Middleware');
      insert zone;
      Project__c project = TestObjectFactory.CreateProject('Project23');
      insert project;
      Catalog_Configuration__c config = TestObjectFactory.CreateCatalogConfig();
      insert config;
      Test.startTest();
      Test.setMock(HttpCalloutMock.class, new StrataApiMock(StrataApiMock.method.PUBLISH, StrataApiMock.status.SUCCESS));
      String result = StrataAPIAccess.Publish(project.id);
      project = [Select catalog_certification_id__c, catalog_URL__c, catalog_sync_status__c, catalog_sync_message__c, catalog_Status_Code__c,certification_Status__c from Project__c where id = :project.id limit 1];
      System.AssertEquals('Success', result,'result should have been success');
      System.AssertEquals('OK',project.catalog_sync_message__c, 'The message should have been OK');
      System.AssertEquals(200, project.catalog_Status_Code__c, 'The status code should have been 200');
      System.AssertEquals('Certification Completed',project.certification_Status__c,'Should be Certification Completed');

      Test.stopTest();
   }

}
