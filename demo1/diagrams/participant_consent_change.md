http://www.plantuml.com/plantuml/uml/pPB1Yjim48RlUeeXzx3tWBjRbhAKik2ksriXgD8a7cBBHYEbk4zVMR68d3ffwAMz62DzC_FFXvxOIUjysmYfN6TXzOQCqgeQod1EYdfft0eaW-r5Vpw2rKTTndFI5nUVS80tREG05TeZAOpOmV8SU-uCet4pIB4GeYZWlNSrci1AHvs1eX32zh1-Dmw33LJ-UqkEBp4okywsyL-Cu3eKs96tg92E-5k1wmveJXDJckTQcZt6YU7q1HsyIdPe4y1PcB3I1bu-VDd0GnkXhPgWRmP-aAlZl6bAW5CDa1v3x0WPSYMUoUn1BjUKQCVjEDvtbdx65Hv5WbHWFdkqOzC0pXfFiMQ2LAXAlK_f4yQnvsHluNBoKZclXpzeIKMFP5JZDHOSYWZ_DpySLqbXSpNBJh9JsJXuxJJjXWjHGW1aFiglzpqJWByH0N2FYZEchYKAoCs70N3pQM33c7HKkLIng2CLFaZTajbcAsv9htbi47x-kQKiQCnqBUXStZLhkIwZnLFu-BRv1G00

@startuml
actor "Research Participant" as rp
participant "IdP" as idp
participant "Key Relay Service" as krs
participant "Consent Service" as cs


== Authentication ==

rp -> idp : Submits Authentication information
idp --> rp : receives Authentication token


== Consent Change ==

== Get Master Consents ==

rp -> krs: GET: /master_consents\n with Auth token
krs -> krs: identifies participant using auth token
krs -> krs: Performs authorization?

krs -> cs: Queries for participant consent information
cs --> krs: master consents

krs --> rp: master consents

== Get Study Consents ==

rp -> krs: GET: /consents?include=studies\n with Auth token
krs -> krs: identifies participant using auth token
krs -> krs: Performs authorization?

krs -> cs: Queries for participant study consents
cs --> krs: consents

krs --> rp: consents

alt Modify master consent

  rp -> krs: PUT: /master_consents\n with Auth token
  krs -> krs: identifies participant using auth token
  krs -> krs: Performs authorization?

  krs -> cs: PUT: /master_consents
  cs --> krs: master consents

  krs --> rp: master consents

else Modify Study consent


  rp -> krs: PUT: /consents/admin_participant_id-study_id\n with Auth token
  krs -> krs: identifies participant using auth token
  krs -> krs: Performs authorization?

  krs -> cs: PUT: /consents/study_participant_id-study_id
  cs --> krs: consents

  krs --> rp: consents

end
@enduml
