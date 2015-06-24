trigger LimitCases on Case (before insert) {
	Integer limitCases = null;
    CaseSettings__c settings = CaseSettings__c.getValues('default');
    
    if (settings != null) {
        limitCases = Integer.valueOf(settings.LimitCases__c);
    }
    
    if (limitCases != null) {
        Set<Id> userIds = new Set<Id>();
        Map<Id, Integer> caseCountMap = new Map<Id, Integer>();
        
        for (Case c: trigger.new) {
            userIds.add(c.OwnerId);
            caseCountMap.put(c.OwnerId, 0);
        }
        
        Map<Id, User> userMap = new Map<Id, User>([
            select Name
            from User
            where Id in :userIds
        ]);
        
        for (AggregateResult result: [
            select count(Id),
            	OwnerId
            from Case
            where CreatedDate = THIS_MONTH and
            	OwnerId in :userIds
            group by OwnerId
        ]) {
            caseCountMap.put((Id) result.get('OwnerId'), (Integer) result.get('expr0'));
        }
        
        for (Case c: trigger.new) {
            
            
            if (caseCountMap.get(c.OwnerId) > limitCases) {
                c.addError('Too many cases created this month for user ' + userMap.get(c.OwnerId).Name + '(' + c.OwnerId + '): ' + limitCases);
            }
          
           else {caseCountMap.put(c.OwnerId, caseCountMap.get(c.OwnerId) + 1);}
        }
    }
}

