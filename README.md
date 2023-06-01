# Gravity Eligibility

## Discover available CDS services
Make a request to CDS Discovery endpoint:
```
GET CDS_SERVICE_BASE_URL/cds-services

200 
{
  "services": [
    {
      "hook": "patient-view",
      "title": "SDOH Service Eligibility Check",
      "description": "Check for availability of SDOH services",
      "id": "sdoh-eligibility-check",
      "prefetch": {
        "patient": "Patient/{{context.patientId}}",
        "coverage": "Coverage?patient=Patient/{{context.patientId}}",
        "goals": "Goal?patient=Patient/{{context.patientId}}&lifecycle-status=active",
        "observations": "Observation?patient=Patient/{{context.patientId}}&category=social-history"
      }
    }
  ]
}
```

## Call CDS service
When viewing a patient's chart, call the CDS service to check for available SDOH
services:

```
POST CDS_SERVICE_BASE_URL/sdoh-eligibility-check
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
    "observations": { "resourceType": "Bundle", "type": "searchset" ... }
  }
}
```

## Receive response
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

### Create a task
A card can be used to create a Task in the user's system (CRD specifically
defines [cards for creating Tasks for the completion of a
Questionnaire](http://www.hl7.org/fhir/us/davinci-crd/hooks.html#request-form-completion),
and this is a modification of that use).

```
200
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
          "decsription": "Create referral to transport company ABC",
          "resource": {
            "resourceType": "Task",
            "for": {
              "reference": "Patient/456"
            },
            ...
          }
        }
      ]

    }
  ]
}
```
