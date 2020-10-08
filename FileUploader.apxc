public class FileUploader {
  private ApexPages.StandardController std;
  public string nameFile{get;set;}
  public Blob contentFile{get;set;}
  public Date closeDate {get;set;}
  public Id recordType {get;set;}
  public Id ownerId {get; set;}
  public List<opportunity> opplist{get;set;}
  public Contact con {get; set;}
  public Opportunity opp {get; set;}
    
  private final String integrationName = 'Inquiry Import';
  String campaignlookup;
  String[] filelines = new String[]{};
        
  public List<SelectOption> repSelectOptions {get {
    List<SelectOption> repSelectOptions = new List<SelectOption> {new SelectOption('','Select One')};
    for (User u : [SELECT Id, Name
                  FROM User 
                  WHERE UserType = 'Standard' AND 
                  IsActive = true
                  ORDER BY Name asc]) {
          repSelectOptions.add(new SelectOption(u.Id, u.Name));
    }
    return repSelectOptions;
  }}
    
  public List<SelectOption> recordOptions {get {
    List<SelectOption> recordOptions = new List<SelectOption>{new SelectOption('','Select One')};
        
    for (RecordType r : [SELECT Id, Name FROM RecordType WHERE sobjecttype = 'Opportunity']) {
      recordOptions.add(new SelectOption(r.Id, r.Name));
    }
    return recordOptions;
  }}
    
  {
    initializeObjects();
  }
            
  public void initializeObjects() {
    con = new Contact();
    opp = new Opportunity();
  }
    
  public FileUploader() {
		opplist = New List<Opportunity>(); 
	}
    
  /***This function reads the CSV file and inserts records into objects. ***/
  public Pagereference ReadFile()
  {
    Map<Integer,Contact> conMap = new Map<Integer,Contact>();
    Map<Integer,Opportunity> oppMap = new Map<Integer,Opportunity>();
    Map<Integer,Integration_Origin__c> originMap = new Map<Integer,Integration_Origin__c>();
    Map<String,Campaign> camMap = new Map<String,Campaign>();
    Map<Integer,String> camNameMap = new Map<Integer,String>();
    set<String> campaignlookupset = new set<String>();
    Integration__c integ = [SELECT Id FROM Integration__c WHERE Name = :this.integrationName limit 1];
    opplist.clear();
    try{
      //Convert the uploaded file which is in BLOB format into a string
      nameFile =blobToString( contentFile,'ISO-8859-1');
                
      //Now sepatate every row of the csv file
      filelines = nameFile.split('\n');
               
      //Iterate through every line and create a Record for each row
      for (Integer i=1; i<filelines.size(); i++)
      {                                       
        String[] inputvalues = new String[]{};
        inputvalues = filelines[i].split(',');
                   
        con.FirstName = inputvalues[0] ;             
        con.LastName = inputvalues[1];
        con.MobilePhone = inputvalues[2];
        con.Personal_Email__c = inputvalues[3];
        con.OwnerId = this.ownerId;
        con.Integration__c = this.integrationName;
        conMap.put(i, con);
                 	
        opp.CloseDate = this.closeDate;
        opp.RecordTypeId = this.recordType;
        opp.OwnerId = this.ownerId;
        if (inputvalues[4] == null) {
          opp.Name = inputvalues[0] + ' ' + inputvalues[1];
        }
        else {
          opp.Name = inputvalues[0] + ' ' + inputvalues[1] + ' - ' + inputvalues[4];
        }
        opp.Program_of_Interest__c = inputvalues[4];
        campaignlookup = inputvalues[5];
        camNameMap.put(i,campaignlookup);
        campaignlookupset.add(campaignlookup);
        opp.StageName = 'New';
        opp.Integration__c = this.integrationName;
        oppMap.put(i,opp);
        opplist.add(opp);
                 
        Integration_Origin__c origin = new Integration_Origin__c();
        origin.Integration__c = integ.Id;
        originMap.put(i,origin);
                 
        initializeObjects();
        campaignlookup = null;
      }
            	
      if (!campaignlookupset.IsEmpty()) {
        for (Campaign cam : [SELECT Id, Name FROM Campaign WHERE Name IN: campaignlookupset]) {
          camMap.put(cam.Name, cam);
        }
      }
            
      if (!conMap.IsEmpty()) {
        insert conMap.Values();
                
        for (Integer i = 1; i <= oppMap.Values().size(); i++) {
          if (conMap.containsKey(i)) {
            oppMap.get(i).Contact__c = conMap.get(i).Id;
                        
            String campName = camNameMap.get(i);
            if (camMap.containsKey(campName)) {
              oppMap.get(i).CampaignId = camMap.get(campName).Id;
            }
          }
        }
                
        if (!oppMap.IsEmpty()) {
          insert oppMap.Values();
          for (Integer i = 1; i <= originMap.Values().size(); i++) {
            originMap.get(i).Contact__c = conMap.get(i).Id;
            originMap.get(i).Opportunity__c = oppMap.get(i).Id;
          }
          if (!originMap.IsEmpty()) {
            insert originMap.Values();
          }
        }
      }
    }
    catch(Exception e){
      initializeObjects();
      ApexPages.Message errormsg = new ApexPages.Message(ApexPages.severity.ERROR,'An error has occured reading the CSV file '+e.getMessage());
      ApexPages.addMessage(errormsg);
    }               
    return null;
  }
    
  public List<Opportunity> getuploadedOpp () {
    return opplist;
  }
  
  public static String blobToString(Blob input, String inCharset){
    String hex = EncodingUtil.convertToHex(input);
    System.assertEquals(0, hex.length() & 1);
    final Integer bytesCount = hex.length() >> 1;
    String[] bytes = new String[bytesCount];
    for(Integer i = 0; i < bytesCount; ++i)
      bytes[i] =  hex.mid(i << 1, 2);
    return EncodingUtil.urlDecode('%' + String.join(bytes, '%'), inCharset);
  }
}
