User
Write a new unit test method and check what is the status of the unit test run + the returnModel attribute values? from this : 

public static Appt_SiteAvailabilityModel getAvailabilityModel( List<Appt_Time_Slot__c> allTimeSlots, Appt_workTypeModel wrkTypModel, String storeWebChar, String fleetRetailChar, DateTime webCutoffTime, Integer timeZoneOffset, String timeZone)
    {
        Datetime startTime = Datetime.now();
        system.debug(' startTime ::: '+startTime);
 
        Datetime methodTime = Datetime.now();
        Appt_SiteAvailabilityModel avModel = new Appt_SiteAvailabilityModel();
        avModel.timeZone = timeZone;
    
        if (wrkTypModel.type == Appt_ConstantValues.WORK_TYPE_TYPE_FRONT) {
 
            //---Get the Front Appointments
            List<Appt_Time_Slot__c> frontList = getAvailableSlots(allTimeSlots, Appt_ConstantValues.APPOINTMENT_TYPE_FRONT_CHAR, storeWebChar, fleetRetailChar, webCutoffTime, timeZoneOffset, avModel);
            System.debug(lprefix+'        getAvailabilityModel() - frontList: ' + frontList.size()  + ' records');
 
        } else if (wrkTypModel.type == Appt_ConstantValues.WORK_TYPE_TYPE_BACK) {
 
            //---Get the Back Appointments
            List<Appt_Time_Slot__c> backList = getAvailableSlots(allTimeSlots, Appt_ConstantValues.APPOINTMENT_TYPE_BACK_CHAR, storeWebChar, fleetRetailChar, webCutoffTime, timeZoneOffset, avModel);
            System.debug(lprefix+'        getAvailabilityModel() - backList: ' + backList.size() + ' records');
 
        } else {
             List<Appt_Time_Slot__c> frontList = getAvailableSlots(allTimeSlots, Appt_ConstantValues.APPOINTMENT_TYPE_FRONT_CHAR, storeWebChar, fleetRetailChar, webCutoffTime, timeZoneOffset, null);
             System.debug(lprefix+'        getAvailabilityModel() - frontList: ' + frontList.size()  + ' records');
             
             List<Appt_Time_Slot__c> backList = getAvailableSlots(allTimeSlots, Appt_ConstantValues.APPOINTMENT_TYPE_BACK_CHAR, storeWebChar, fleetRetailChar, webCutoffTime, timeZoneOffset, null);
             System.debug(lprefix+'        getAvailabilityModel() - backList: ' + backList.size() + ' records');
 
             //---For performance, need to put the back list into a Map.  This only scans the back list once, and then allows very fast searching by the key (StartTime)
             Map<String, Appt_Time_Slot_c> backMap = new Map<String, Appt_Time_Slot_c>();
             
             for (Appt_Time_Slot__c currSlot : backList)
             {
                 String key = '' + currSlot.Start_Time__c;   //---This creates to a String, this assumes that the date is included in the String
                 //System.debug(lprefix+'        putting on backMap, key='+key+', currSlot:'+currSlot);
 
                 backMap.put( key, currSlot);
             }
           
            //---For each Front Appointment, check if there is a matching Back Appointment
            for (Appt_Time_Slot__c frontSlot : frontList) 
            {
                String key = '' + frontSlot.End_Time__c;   //---This creates to a String, this assumes that the date is included in the String
                //System.debug(lprefix+'        checking on backMap for cooresponding frontslot, key='+key+', currSlot:'+frontSlot);
 
                Appt_Time_Slot__c backSlot = backMap.get(key);
                if (backSlot != null) addTimeSlotsToModel( avModel, frontSlot, backSlot);
            }
        }
        
        Datetime endTime = Datetime.now();
          system.debug(' EndTime ::: '+endTime);
          System.debug('Difference time in Millisecond: ' + (endTime.getTime() - startTime.getTime()));
          System.debug('Difference time in Seconds: ' + (endTime.getTime() - startTime.getTime())/ 1000);
          system.debug('CPU USAGE :: '+Limits.getCPUTime());
        System.debug( lprefix + 'M:getAvailabilityModel(): ' + Fleet_IntegrationUtil.getDuration(methodTime, Datetime.now()) + ' ms');
        return avModel;
    }
