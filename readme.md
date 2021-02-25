# POSTMAN x Testlink integration

A simple script to enable POSTMAN and Testlink integration

## Requirement

* [POSTMAN](https://www.postman.com/downloads/)
* Already know Project ID, Testplan ID, Build ID, and Platform ID of application/project under test

## Usage

Demo can be seen by importing [DEMO.zip](DEMO.zip) to POSTMAN. If you want to enable POSTMAN and Testlink integration to an existing collection:

1. Add post-test script to collection
```javascript
testFunctions = {
    setResult: function(tcid, expectedToBeOk, additionalResult = [true], addLog = null){
        if (expectedToBeOk === true){
            pm.test(pm.response.text(), function() { pm.response.to.be.ok});
            if (pm.response.code == 200){
                result = 'p';
            } else {
                result = 'f';
            }
            
        }
        else{
            pm.test(pm.response.text(), function() { pm.response.to.be.not.ok});
            if (pm.response.code !== 200){
                result = 'p';
            } else {
                result = 'f';
            }
        }
        
        message = pm.response.status;
        
        if (typeof(additionalResult) !== 'object'){additionalResult = [additionalResult]}
        addResult = true
        for (var item of additionalResult){
            addResult = addResult && item;
        }
        
        
        if (result === 'p' && addResult){
            result2 = 'p';
        } else {
            result2 = 'f';
        }
        
         if (result2 !== 'p'){
            try{
                message += ' | ' + JSON.stringify(pm.response.json());
                message = addLog + message;
            } catch(err){
                message += ' | ' + pm.response.text();
                message = addLog + message
            }
        }
        
        //console.log("ASSIGNING TL RESULT: " + tcid + " | " + result2 + " | " + message);
        console.log(addLog);
        var pmRequestParams = {
            url: pm.variables.get('TL_URL'),
            method: 'POST',
            header: {
                'Content-Type': 'text/xml'
            },
            body: {
                mode: 'raw',
                raw: '<?xml version=\'1.0\'?><methodCall><methodName>tl.reportTCResult</methodName><params><param><value><struct><member><name>buildid</name><value><string>' +pm.variables.get('TL_BUILDID') + '</string></value></member><member><name>testcaseexternalid</name><value><string>'+tcid+'</string></value></member><member><name>platformid</name><value><string>'+ pm.variables.get('TL_PLATFORMID') +'</string></value></member><member><name>devKey</name><value><string>'+ pm.variables.get('TL_DEVKEY') +'</string></value></member><member><name>testcaseid</name><value><nil/></value></member><member><name>testplanid</name><value><string>'+ pm.variables.get('TL_TESTPLANID') +'</string></value></member><member><name>buildname</name><value><nil/></value></member><member><name>status</name><value><string>'+result2+'</string></value></member><member><name>notes</name><value><string>'+message+ '</string></value></member></struct></value></param></params></methodCall>'
                }
            };
            
        if (pm.variables.get('TL_ASSIGNRESULT').toLowerCase() == "true" && pm.variables.get("HOST") !== 'localhost'){
            return pmRequestParams;
        } else if (pm.variables.get('TL_ASSIGNRESULT').toLowerCase() == 'log') {
            console.log(pmRequestParams);
            return null;
        } else {
            return null;
        }
    }  
}

```
2. Add variables to collection
   * TL_DEVKEY: Personal API key
   * TL_PROJECTID
   * TL_TESTPLANID
   * TL_BUILDID
   * TL_PLATFORMID
   * TL_ASSIGNRESULT: false/log/true
   * TL_URL: Testlink xmlrpc URL e.g. http://localhost/lib/api/xmlrpc/v1/xmlrpc.php

3. Add post-test script to test script
```javascript
pm.sendRequest(testFunctions.setResult(fullTestCaseId, expectedResult, additionalAssertionResult))
```
   where:
   * fullTestCaseId: string
   
      Full test case ID related to test case in TestLink e.g. AUTO-213, ANY-2348, etc

   * expectedResult: boolean

      Basic assertion as expected result. Set to "true" if 200 response code is expected, or set to "false" if other than 200 response code is expected.

   * additionalAssertionResult: array of boolean, optional

      Additional assertion result to complement basic assertion.

## Example post-test script
```javascript
//expecting test case "OK-86" to have 200 as response code
pm.sendRequest(testFunctions.setResult("OK-86", true)

//expecting test case "NOK-123" to have non 200 as response code
pm.sendRequest(testFunctions.setResult("NOK-123", false)

//expecting test case "FRBDN-403" to have 403 response code and response time under 200ms
result1 = pm.response.code === 403
result2 = pm.response.responseTime < 200
pm.sendRequest(testFunctions.setResult("FRBDN-123", false, [result1, esult2])

//if test script title is in "testCaseID testCaseTitle" format, use request.name.split(" ")[0] to get testCaseID automatically from test script title
pm.sendRequest(testFunctions.setResult(request.name.split(" ")[0], true))

```

Further example can be seen in [DEMO.json](DEMO.json).