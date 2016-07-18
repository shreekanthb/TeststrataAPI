public with sharing class StrataAPI {
    public enum Instance {PROD, DEV, STAGE, QA}
    public StrataAPi()
    {
        Catalog_Configuration__c config = [Select namespace__c, endpoint__c, username__c, password__c from Catalog_Configuration__c where active__c = true limit 1];
        namespace = config.namespace__c;
        baseEndpoint = config.Endpoint__c;
        username = config.Username__c;
        password = config.Password__c;
    }
    private string namespace = '';
    private string baseEndpoint = '';
    private string username = '';
    private string password = '';


    public HttpRequest CreateProductRequest(string assetId, string vendorId)
    {
      system.debug('strataapi create req' + vendorId);
        string endpoint = getEndpoint('software','');
        string authorizationHeader = getAuthenicationHeader();
        string content = getCreateProductXML(assetId, vendorId);
        system.debug('content: ' + content);
        system.debug('endpoint: ' + endpoint);
        HttpRequest req = new HttpRequest();
        req.setMethod('POST');
        req.setEndpoint(endpoint);
        req.setHeader('Accept','application/vnd.redhat.certifiedsoftware+xml');
        req.setHeader('Content-Type','application/vnd.redhat.certifiedsoftware+xml');
        req.setHeader('Authorization', authorizationHeader);
        req.setBody(content);
      req.setTimeout(100000);
        return req;
    }

    public HttpRequest UpdateProductRequest(string assetId, string productId, string vendorId)
    {
      //system.debug('strataapi create req' + vendorId);
        string endpoint = getEndpoint('software',productId);
        string authorizationHeader = getAuthenicationHeader();
        string content = getCreateProductXML(assetId, vendorId);
        system.debug('content: ' + content);
        system.debug('endpoint: ' + endpoint);
        HttpRequest req = new HttpRequest();
        req.setMethod('PUT');
        req.setEndpoint(endpoint);
        req.setHeader('Accept','application/vnd.redhat.certifiedsoftware+xml');
        req.setHeader('Content-Type','application/vnd.redhat.certifiedsoftware+xml');
        req.setHeader('Authorization', authorizationHeader);
        req.setBody(content);
      req.setTimeout(100000);
        return req;
    }

    public HttpRequest CreateCertificationRequest(string projectId, string productId)
    {
        string content = getCreateCertificationXML(projectId);
        string authorizationHeader = getAuthenicationHeader();
        string endpoint = getEndPoint('software',productId);
        endpoint = endpoint +  '/certifications/';
        HttpRequest req = new HttpRequest();
        req.setMethod('POST');
        req.setEndpoint(endpoint);
        req.setHeader('Content-Type','application/vnd.redhat.certification+xml');
        req.setHeader('Authorization', authorizationHeader);
        req.setBody(content);
      req.setTimeout(100000);
      System.debug('content' + content);
      System.Debug('request: ' + req.tostring());
            System.debug('endPoint:' + endpoint);
        return req;
    }

    public HttpRequest UpdateCertificationRequest(string projectId, string productId, string certId)
    {
        string content = getCreateCertificationXML(projectId);
        string authorizationHeader = getAuthenicationHeader();
        string endpoint = getEndPoint('software',productId);
        endpoint = endpoint +  '/certifications/' + certId + '/';
        HttpRequest req = new HttpRequest();
        req.setMethod('PUT');
        req.setEndpoint(endpoint);
        req.setHeader('Content-Type','application/vnd.redhat.certification+xml');
        req.setHeader('Authorization', authorizationHeader);
        req.setBody(content);
      req.setTimeout(100000);
      System.debug('content' + content);
      System.Debug('request: ' + req.tostring());
            System.debug('endPoint:' + endpoint);
        return req;
    }
    public HttpResponse Publish(string projectId)
    {
        Project__c project = [SELECT Partner_Product_Version__r.catalog_visibility__c,product_ref__r.catalog_product_id__c, certification_Status__c FROM Project__c where id = :projectId];
        string endpoint = getEndpoint('software',project.product_ref__r.catalog_product_id__c);
        string authorizationHeader = getAuthenicationHeader();
        string content = getPublishXml();
        HttpRequest req = new HttpRequest();
        req.setMethod('PUT');
        req.setEndpoint(endpoint);
        req.setHeader('Accept','application/vnd.redhat.certifiedsoftware+xml');
        req.setHeader('Content-Type','application/vnd.redhat.certifiedsoftware+xml');
        req.setHeader('Authorization', authorizationHeader);
        req.setBody(content);
        req.setTimeout(100000);
        Http http = new Http();
        HttpResponse response = http.send(req);
        Integer statusCode = response.getStatusCode();
        if (statusCode == 200 || statusCode == 201)
        {
         project.partner_Product_Version__r.catalog_visibility__c = true;
         project.certification_Status__c = 'Certification Completed';
         update project;
        }
      return response;
    }


    public HttpResponse getResponse(HttpRequest req)
    {
        Http http = new Http();
        HttpResponse res = http.send(req);
        return res;
    }

    public String getCatalogId(HttpResponse response)
    {
        return getNodeText(response.getBody(), 'id');
    }

    public String getViewUri(HttpResponse response)
    {
        return getNodeText(response.getBody(),'viewUri');
    }


   private string getCreateProductXML(string assetId, string vendorId)
   {
      Asset product = [SELECT Link_to_UCC__c,Name,Text_Product_Description__c,Webpage__c,Product_Type_Category__r.name,Brief_Overview__c FROM Asset where id = :assetId];
      List<Partner_Product_Versions__c> prodVersions = [SELECT Availability__c,Catalog_visibility__c, Version__c FROM Partner_Product_Versions__c where partner_product__c = :assetId];
      XmlStreamWriter writer = new XmlStreamWriter();
      writer.writeStartDocument('UTF-8','1.0');
      writer.writeStartElement(null, 'certifiedSoftware', null);
      writer.writeDefaultNamespace(namespace);
      writer.writeStartElement(null,'title', null);
      writer.writeCharacters(product.Name);
      writer.writeEndElement();
      if (prodVersions.size() > 0 && String.isNotBlank(prodVersions[0].Version__c))
      {
         writer.writeStartElement(null,'productVersion', null);
         writer.writeCharacters(prodVersions[0].Version__c);
         writer.writeEndElement();
      }
      if (string.isNotBlank(product.brief_Overview__c))
      {
           writer.writeStartElement(null,'productSummary',null);
           writer.writeCharacters(product.brief_Overview__c);
           writer.writeEndElement();
       }
       if (string.isNotBlank(product.text_Product_Description__c))
       {
           writer.writeStartElement(null,'productDescription',null);
           writer.writeCharacters(product.text_Product_Description__c);
           writer.writeEndElement();
        }
        if (string.isNotBlank(product.Webpage__c))
        {
           writer.writeStartElement(null,'productUrl',null);
           writer.writeCharacters(product.Webpage__c);
           writer.writeEndElement();
        }

        if (prodVersions.size() > 0)
        {
           writer.writeStartElement(null,'isAvailable',null);
           writer.writeCharacters(convertBoolean(prodVersions[0].Availability__c));
           writer.writeEndElement();
        }
        else
        {
            writer.writeStartElement(null,'isAvailable',null);
           writer.writeCharacters('false');
           writer.writeEndElement();
        }
        if (string.isNotBlank(vendorId))
        {
           writer.writeStartElement(null,'vendor',null);
               writer.writeStartElement(null,'id',null);
               writer.writeCharacters(vendorId);
               writer.writeEndElement();
           writer.writeEndElement();
        }
           writer.writeStartElement(null,'categories',null);
               writer.writeStartElement(null,'value',null);
               writer.writeCharacters(product.Product_Type_Category__r.name);
               writer.writeEndElement();
           writer.writeEndElement();
      writer.writeEndElement();
      return writer.getXmlString();
   }

  private string getPublishXml()
  {
     XmlStreamWriter writer = new XmlStreamWriter();
     writer.writeStartDocument('UTF-8','1.0');
     writer.writeStartElement(null, 'certifiedSoftware', null);
     writer.writeDefaultNamespace(namespace);
     writer.writeStartElement(null,'published',null);
     writer.writeCharacters('true');
     writer.writeEndElement();
     writer.writeEndElement();
     return writer.getXmlString();
 }

    private string getCreateCertificationXML(string projectId )
    {
      Project__c project = [SELECT RedHat_Product__r.name,Red_Hat_Product_Version__r.version__c, Partner_Product_Version__r.Version__c,Red_Hat_Product_Version__r.Minor_version__c,Red_Hat_Product_Version__r.Name,Zone__r.Name FROM Project__c where id = :projectId];
        XmlStreamWriter writer = new XmlStreamWriter();
        writer.writeStartDocument('UTF-8','1.0');
        writer.writeStartElement(null, 'certification', null);
        writer.writeDefaultNamespace(namespace);
        if (String.isNotBlank(project.RedHat_Product__r.name))
        {
           writer.writeStartElement(null,'productName',null);
            // writer.writeStartElement(null,'value',null);
            writer.writeCharacters(project.RedHat_Product__r.name);
             //writer.writeEndElement();
            writer.writeEndElement();
        }
        if (String.isNotBlank(project.Partner_Product_Version__r.Version__c))
        {
            writer.writeStartElement(null,'productVersion',null);
            writer.writeCharacters(project.Partner_Product_Version__r.version__c);
            writer.writeEndElement();
         }
        if (String.isNotBlank(project.Red_Hat_Product_Version__r.version__c))
        {
            writer.writeStartElement(null,'productMajorVersion',null);
            writer.writeCharacters(project.Red_Hat_Product_Version__r.version__c);
            writer.writeEndElement();
         }
        //The catalog expect the minor version to inculde major version.  The name is major and minor version concatenated.  If there is not a minor version,
        //then major version will be put into minor verion.
        if (String.isNotBlank(project.Red_Hat_Product_Version__r.minor_version__c))
        {
            writer.writeStartElement(null,'productMinorVersion',null);
            writer.writeCharacters(project.Red_Hat_Product_Version__r.Name);
            writer.writeEndElement();
        }
        else
        {
            writer.writeStartElement(null,'productMinorVersion',null);
            writer.writeCharacters(project.Red_Hat_Product_Version__r.version__c);
            writer.writeEndElement();
        }

        if (String.isNotBlank(project.Zone__r.Name))
        {
            if (project.Zone__r.Name.equals('RHEL') || project.Zone__r.Name.equals('Middleware') )
            {
                writer.writeStartElement(null,'certificationLevel', null);
                writer.writeCharacters('Self-Certified');
                writer.writeEndElement();

                writer.writeStartElement(null,'workflowstatus',null);
                writer.writeCharacters('Completed');
                writer.writeEndElement();
            }
            else
            {
                writer.writeStartElement(null,'certificationLevel', null);
                writer.writeCharacters('Certified');
                writer.writeEndElement();

                writer.writeStartElement(null,'workflowstatus',null);
                writer.writeCharacters('In Progress');
                writer.writeEndElement();
            }

        }
             writer.writeStartElement(null,'moderationState',null);
             writer.writeCharacters('draft');
             writer.writeEndElement();
        writer.writeEndElement();
        return writer.getXmlString();
    }

    private string getNodeText(string xmlBody, string name)
    {
        string text = '';
        Dom.Document doc = new Dom.Document();
        try {
            doc.load(xmlBody);
            for (dom.XmlNode node : doc.getRootElement().getChildElements())
            {
                if (node.getName().equals(name))
                {
                    text = node.getText();
                }
            }
        } catch (Exception e)
        {
            System.debug('failure to parse XML: ' + e.getMessage());
        }
        return text;
    }

    private string getEndpoint(string suffix, string appendId)
    {
        string newEndpoint = baseEndpoint + suffix;
        //string endpoint = 'https://api.access.redhat.com/rs/ecosystem/' + suffix;
        //string endpoint = 'https://api.access.stage.redhat.com/rs/ecosystem/' + suffix;
        //string endpoint = 'https://209.132.177.20/rs/ecosystem/software';
        if (string.isnotBlank(appendId))
        {
            newEndpoint += '/' + appendId;
        }
        return newEndpoint;
    }
    private string getAuthenicationHeader()
    {
        Blob headerValue = Blob.valueOf(username + ':' + password);
        String authorizationHeader = 'Basic ' + EncodingUtil.base64Encode(headerValue);
        return authorizationHeader;
    }

    private string convertBoolean(Boolean value)
    {
        string boolString = '';
        if (value)
            boolString = 'true';
        else
            boolString = 'false';
        return boolString;
    }
}
