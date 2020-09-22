For use in:
http://www.plantuml.com/plantuml/uml/rPHDZzem48Rl_XKZxWZKOwIqzAD25QeQ0Jsr5NBjIN59xCXsM6KH_tqTux31YyXbjTgz8ViPlsyUZPWPIxMjou9GPfM6qH8DKissaBbpmKH5fHq1DQ0hKZqUmUWRp_ovuD750XjOOa4RMA7U2uOUssbpYNrMqg2q5n0pX90qXO-rtQ9qBBL2IKXJGdG5u_Tj62Imgl-MmfeO4p9NklI_IGkEtO1kUOqCnHwV35YrG-a3vsZX2-QqBYo4OgONwyedCF-wJc32JzI6TTHlUIVirZyT7dJqUKZmDYyNQA3_zRKHyX_GwpCPwlP3ZBzNMjpxjXz81x5J6PZqNZIJWx4rRpRdG7sCknBcnhkGTzCf_5YuzfyKRn88A0JKPmM1msmGc6dr7zcGmCfJRqDWtFMMmLy1XWx-4qiSn9t7xugBl8btCJQP_Lo_casCkyIYRkzMVDcN4FRffeJXRJeRFtGihQeY_0Li13--aALdNhV0NfSwmCvGyhei7W00

@startuml
actor "Research Participant" as rp
participant "IdP" as idp
participant "Key Relay Service" as krs
participant "Consent Service" as cs


== Authentication ==

rp -> idp : Submits Authentication information
idp --> rp : receives Authentication token

== Get Default Consents ==

rp -> krs: GET: /default_consents \nwith Auth token
krs -> krs: identifies participant using auth token
krs -> krs: Performs authorization?

krs -> cs: GET /participants/{study_identifier}/default_consent
cs --> krs: 200 OK \nBody: default consent

krs --> rp: 200 OK \nBody: default consent

== Get Project Consents ==

rp -> krs: GET: /project_consents \nwith Auth token
krs -> krs: identifies participant using auth token
krs -> krs: Performs authorization?

krs -> cs: GET /participants/{study_identifier}/project_consents
cs --> krs: 200 OK \nBody: [project consents]

krs --> rp: 200 OK \nBody: [project consents]

== Modify Default Consent ==

rp -> krs: PUT /default_consents \nwith Auth token
krs -> krs: identifies participant using auth token
krs -> krs: Performs authorization?

krs -> cs: PUT /default_consents
cs --> krs: default consent

krs --> rp: default consent

== Modify Project Consent ==

rp -> krs: PUT /project_consents \nwith Auth token \nBody: {project_application_id}
krs -> krs: identifies participant using auth token
krs -> krs: Performs authorization?

krs -> cs: PUT /participants/{study_identifier}/project_consents
cs --> krs: 200 OK \nBody: project consent

krs --> rp: 200 OK \nBody: project consent
@enduml
