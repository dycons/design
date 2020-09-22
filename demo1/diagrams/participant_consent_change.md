For use in:
http://www.plantuml.com/plantuml/uml/pPDFQzj04CNl-oa6drf8SEZHu3RzKq88fS99Jqq9czrHDBMqg-wENDIGxzwHMCcn71mAXVPcDBytR-Rj-3Io3jnKfTA6VO3P2YFgO0h8v3iPghNZ6UW8eLRrzXFCpcrsxLvSO6jAhzSlUuRi198ohB3KBLnYs9317bk7k8kzzowYuxX3cQYKMYxXRSC5Ee4ratUmN2eLQZX-tRd10YwRsueuJZ5okGzL_rjruB48GiUlq21jS4_0VeskDpn3Xd6YhPrClO2pZrSmDqFnslqaBlfEIUyy8mIwArX13zzwgLdCTLCoQ6UX3luxcguyjY47tXdHuOy68nz9ZJr4lK5Wgb2XoBh6IaOO9yamT1j6qEBk0_tgrtYpmvpRRi6SCCBZl9j8xWxqEBNiaIERGh3xo8rjRsWGXWSrT3J3-ezQnVc-O9wslKdxAIFoOQ9N9oVm-K8CttdRBiPW4lhdNYdDmIiUClkQu37uRyclFutdPZjwKY_dyibRobluaF1qETF3oRvSmPZCWGFwvq-yF-WqraamEo7k5xS9PIc-GmcbY0yaDBxF_S3gkYxJlrYsUlXF8loRbvlOfn9_bZr5Pvkg_0q0

@startuml
actor "Research Participant" as rp
participant "IdP\n(Keycloak)" as idp
participant "Key Relay Service" as krs
participant "Consents Service" as cs


== Authentication ==

rp -> idp : Submits Authentication information
idp --> rp : Receives Authentication token

== Get Consents ==

rp -> krs: GET /consents \nwith Auth token
note right
  GET both default and project
  consents in once request,
  to save time
end note
krs -> krs: Performs authorization?
krs -> krs: Identifies participant using auth token \nfetches {study_identifier}

krs -> cs: GET /participants/{study_identifier}/default_consent
cs --> krs: 200 OK \nBody: default consent

krs -> cs: GET /participants/{study_identifier}/project_consents
cs --> krs: 200 OK \nBody: [project consents]

krs --> rp: 200 OK \nBody: default consent, [project consents]

== Modify Default Consent ==

rp -> krs: PUT /default_consents \nwith Auth token
krs -> krs: Performs authorization?
krs -> krs: Identifies participant using auth token \nfetches {study_identifier}

krs -> cs: PUT /default_consents
cs --> krs: default consent

krs --> rp: default consent

== Modify Project Consent ==

rp -> krs: PUT /project_consents \nwith Auth token \nBody: {project_application_id}
krs -> krs: Performs authorization?
krs -> krs: Identifies participant using auth token \nfetches {study_identifier}

krs -> cs: PUT /participants/{study_identifier}/project_consents
cs --> krs: 200 OK \nBody: project consent

krs --> rp: 200 OK \nBody: project consent
@enduml