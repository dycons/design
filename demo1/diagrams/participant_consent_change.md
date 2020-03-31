For use in:
http://www.plantuml.com/plantuml/uml/pPBDhjCm44RtVefHzzNTLmejAgX222aqi4EgZ3rfHasSo3z5uUbnt9Ww9IqL6-wcYiPdFD-SUcCirzOrDUF2jXeMUpJ8jQWWztKIr75b5y0Dw8vrrn8iFiWyr4dU77p47lPOynuAr6SI6B2JDbDisog3oauWOHXZMGPldQrGUPPRQXLa6MEwW-MROHYie73V6xBc5YDLjhe9tsm0bxv13xn6WNJ6FnfiUq9rcHZJR2kkZkWdXkeEBRp3ahlQHi8aQVnEAzYz_xA25tcX3cAalYduGROA4ySvdlxTH7A8KH8QIAKuG-e8VBudHptiPy9vg-bdM6BD8YGyyzcX7YxruEJYCTP4aV1TyUunVkHCk7GKE7CQtHJMoVulPcBJcfIedSJC-2R2rsR8X5LYs5JEl9IaohsQJwsaihyrnHX08YJ_-kHJ0VWN5G3FoBXYucuappn-7G2FtmVM1gEJ8ZNBPfN4nXSk6rA7PCi3oMNuQ_p7VnUL10sPxWMTwhkhBT6b9Dlu7zVKlm00

@startuml
actor "Research Participant" as rp
participant "IdP" as idp
participant "Key Relay Service" as krs
participant "Consent Service" as cs


== Authentication ==

rp -> idp : Submits Authentication information
idp --> rp : receives Authentication token


== Consent Change ==

== Get Default Consents ==

rp -> krs: GET: /default_consents\n with Auth token
krs -> krs: identifies participant using auth token
krs -> krs: Performs authorization?

krs -> cs: GET: /default_consents
cs --> krs: default consents

krs --> rp: default consents

== Get project Consents ==

rp -> krs: GET: /project_consents?include=studies\n with Auth token
krs -> krs: identifies participant using auth token
krs -> krs: Performs authorization?

krs -> cs: GET: /project_consents?include=studies\n (this may be multiple calls for distinct objects)
cs --> krs: project consents

krs --> rp: project consents

alt Modify default consent

  rp -> krs: PUT: /default_consents\n with Auth token
  krs -> krs: identifies participant using auth token
  krs -> krs: Performs authorization?

  krs -> cs: PUT: /default_consents
  cs --> krs: default consents

  krs --> rp: default consents

else Modify project consent


  rp -> krs: PUT: /project_consents/admin_participant_id-project_id\n with Auth token
  krs -> krs: identifies participant using auth token
  krs -> krs: Performs authorization?

  krs -> cs: PUT: /consents/project_participant_id-project_id
  cs --> krs: project consents

  krs --> rp: project consents

end
@enduml
