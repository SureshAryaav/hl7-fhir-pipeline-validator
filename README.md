# hl7-fhir-pipeline-validator
hl7-fhir-pipeline-validator
Takes an HL7 v2 message, parses it, and validates that the corresponding FHIR resource has correct mapped data.
public class HL7ToFHIRMappingValidator {

    @Test(description = "Validate PID segment maps correctly to FHIR Patient")
    public void testPatientDemographicsMapping() {
        
        // Simulate what your EHR sends
        String hl7Message = "MSH|^~\\&|EHR|HOSPITAL|eDH|DEST|20240115||ADT^A01|001|P|2.5\n" +
                           "PID|1||MRN12345^^^HOSPITAL^MR||Doe^John^A||19800115|M";
        
        // Parse HL7
        HL7Parser parser = new HL7Parser();
        HL7PatientData hl7Patient = parser.parsePatient(hl7Message);
        
        // What FHIR should have after transformation
        // In real world, you'd query your eDH endpoint
        // Here we use HAPI public server with pre-loaded data
        Response fhirResponse = FHIRClient.searchPatients("identifier", "MRN12345");
        
        // The assertions that matter
        String fhirFamily = fhirResponse.jsonPath()
            .getString("entry[0].resource.name[0].family");
        String fhirBirthDate = fhirResponse.jsonPath()
            .getString("entry[0].resource.birthDate");
        
        assertThat(fhirFamily)
            .as("PID-5 family name should map to FHIR Patient.name.family")
            .isEqualTo(hl7Patient.getFamilyName());
            
        assertThat(fhirBirthDate)
            .as("PID-7 DOB should map to FHIR Patient.birthDate in YYYY-MM-DD format")
            .isEqualTo("1980-01-15");
    }
}
```

**Add a mapping reference document to the repo:**
Create `docs/HL7-to-FHIR-mapping.md` with your field mapping table. This shows you understand the domain deeply, not just the tools.

---

