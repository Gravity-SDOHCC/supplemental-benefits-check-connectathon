# Social Care Supplemental Benefits Check

## Workflow diagram
![Social care supplemental benefits check workflow diagram](https://github.com/Gravity-SDOHCC/social-care-supplemental-benefits/blob/main/gravity_workflow.svg)

## Discover available CDS services (not on workflow diagram)
Make a request to CDS Discovery endpoint:
```
GET CDS_SERVICE_BASE_URL/cds-services

200 
{
  "services": [
    {
      "hook": "patient-view",
      "title": "Social Care Supplemental Benefit Check",
      "description": "Check for available social care supplemental benefits",
      "id": "sdoh-benefit-check",
      "prefetch": {
        "patient": "Patient/{{context.patientId}}",
        "coverage": "Coverage?patient=Patient/{{context.patientId}}&status=active",
        "goals": "Goal?patient=Patient/{{context.patientId}}&lifecycle-status=active&category:in=http://hl7.org/fhir/us/sdoh-clinicalcare/ValueSet/SDOHCC-ValueSetSDOHCategory",
        "conditions": "Condition?patient=Patient/{{context.patientId}}&category:in=http://hl7.org/fhir/us/sdoh-clinicalcare/ValueSet/SDOHCC-ValueSetSDOHCategory",
        "consent": "Consent?patient={{context.patientId}}&category=IDSCL&status=active"
      }
    }
  ]
}
```

## Call CDS service (1 on diagram)
When viewing a patient's chart, call the CDS service to check for available SDOH
services:

```
POST CDS_SERVICE_BASE_URL/sdoh-benefit-check
{
  "hook": "patient-view",
  "hookInstance": "d1577c69-dfbe-44ad-ba6d-3e05e953b2ea",
  "context": {
    "userId": "Practitioner/123",
    "patientId": "456"
  },
  "prefetch": {
    "patient": { "resourceType": "Patient", "id": "456" ... },
    "coverage": { "resourceType": "Bundle", "type": "searchset" ... },
    "goals": { "resourceType": "Bundle", "type": "searchset" ... },
    "conditions": { "resourceType": "Bundle", "type": "searchset" ... },
    "consent": { "resourceType": "Bundle", "type": "searchset" ... }
  }
}
```

## Receive response (2 on diagram)
Several types of responses can be received. Successful responses will have a 200
HTTP status code.

### No relevant information available
Return an empty `cards` array if no relevant information is available for the
patient.

```
200
{
  "cards": []
}
```

### Instructions
[Instructional
cards](http://www.hl7.org/fhir/us/davinci-crd/hooks.html#instructions) present
textual guidance to the user.

```
200
{
  "cards": [
     {
       "summary": "Patient is elegible for non-emergency medical transpont and has used 5/20 available rides",
       "indicator": "info",
       "source": {
          "label": "You're Covered Insurance",
          "url": "https://example.com",
          "icon": "https://example.com/img/icon-100px.png"
        }
     } 
  ]
}
```

### External references
[External reference
cards](http://www.hl7.org/fhir/us/davinci-crd/hooks.html#external-reference)
present links to external resources.

```
200
{
  "cards": [
     {
       "summary": "Enrollment instructions for transportortation assistance",
       "indicator": "info",
       "source": {
          "label": "You're Covered Insurance",
          "url": "https://example.com",
          "icon": "https://example.com/img/icon-100px.png"
        },
        "links": [
          "label": "Transportation Assistance Enrollment Instructions",
          "url": "https://example.com/transportation-enrollment-instructions.pdf"
        ]
     } 
  ]
}
```

### Launch SMART application
[A link to launch a SMART
app](http://www.hl7.org/fhir/us/davinci-crd/hooks.html#launch-smart-application)
can be used to provide custom functionality.

```
200
{
  "cards": [
    {
      "summary": "Launch transportation assistance enrollment app",
      "indicator": "info",
      "source": {
        "label": "You're Covered Insurance",
        "url": "https://example.com",
        "icon": "https://example.com/img/icon-100px.png"
      },
      "links": [
        "label": "Transportation Enrollment",
        "url": "https://example.com/launch-transportation-enrollment",
        "type": "smart"
      ]
    }
  ]
}
```

### Create a ServiceRequest & Task
A card can be used to create a Task in the user's system (CRD specifically
defines [cards for creating Tasks for the completion of a
Questionnaire](http://www.hl7.org/fhir/us/davinci-crd/hooks.html#request-form-completion),
and this is a modification of that use).

```
200
{
  "cards": [
    {
      "summary": "Refer the patient to transportation service provider",
      "indicator": "info",
      "source": {
        "label": "You're Covered Insurance",
        "url": "https://example.com",
        "icon": "https://example.com/img/icon-100px.png"
      },
      "suggestions": [
        {
          "label": "Refer to transport company ABC",
          "actions": [
            {
              "type": "create",
              "decsription": "Create ServiceRequest for referral to transport company ABC",
              "resource": {
                "resourceType": "ServiceRequest",
                "id": "abc123",
                "subject": {
                  "reference": "Patient/456"
                },
                ...
              }
            },
            {
              "type": "create",
              "description": "Create referral task",
              "resource": {
                "resourceType": "Task",
                "focus": {
                  "reference": "ServiceRequest/abc123"
                },
                ...
            }
          ]
        }
      ]
    }
  ]
}
```
