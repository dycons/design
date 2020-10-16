For use in:
http://www.plantuml.com/plantuml/uml/xLLDRzms4BtpLmpsKXjmi6xHqm9ku-OZAD8sWXqVYgO8njHebrsAP4ZAXXhaltSaShMIx6nH0qM1nbVDcpTlFXxrD4JqiQcrG1cjXyK51K8lDr3mToMLGnCNW06y4sx_2HQZE1JMHzGzpCrWFvR5EtFqcbgfBMwFUumgPo0EmmLfRE6I_8sIrCEsFanX2L9OhMGBvsioaQdV5EU9rJgy3yxSbT8q4l5zmc-hAVoPDO4fmqI334A8ipCuR-A638CnAclWx4m8x-39Tzpl2YwRwrfnehPhPK2PogxWdMa2UOCrdO33m9tqS6vzDIQZ0BTB6EduQamZ1Eh8z59I25MZ0ISo130VqsOPCutHRgc3k8JmZ5Zyr5E7WN31EXU_sWPkbTOGBATT4tWgbISPgMGIQ6tZOHZEnjQq4AG3mKjKEa56ikYmgAhFkLTLOsQw2aqO26x8Lwe5D2Lu2lseaoR9EZjgr84LcJ9jwXN5lLhUKnuy7zCANhru2qkPWrpYLiLDhp0FozX8uDLw4mNqu6lBiP8gR7Ji5JXl_o3PXGSQbcwDv3pwq5289noB5WBU44HLKwSCEc8XM4CdnJfSmKL21nZVO8XDsOBgkweKUS7G8QFWELXVzyEsNlsP1cfDTsjtpGX7Dme1dO8jjSSdZ1aLoi6XyUDqQEUPEItLK_IAkYusAinET2GH88lSLYjuIL5knbYkYo5OgJ0oQpVlKLa1d0HFTWr-xDj_l-V-j2FF-xiFmP0yLhbRyAXyMDvDMUPzliyR5C2f-nhVd9x2czUi-mTRjgjX-Hdya7hfPePwuN31tp8STl3VSyt1KeSLddo6mGtFrJEK6mf354gCk0CaHpvNmMbimzJsu71D8dd_5ni17tmkG_D_opDtn32A9FJRqw_Xokm9zi_0pxPKLGlFyt1o8vzvkEUekalEJzRa0Ickti9mFmSUad_TwezMBwx-iTNdrfjRPswqUNZYeYjNOgIHHMHdg-cVxOAqnV9_OACYloY7RJH77RQHI-W3DXgUk8yxB3gdCo7l-PFu3utsQBI7_-Co-dxV7K1_iJQyf12wotnXIZWg6-geewh9DX6idnp7XqR5fRPh3XttFXtz-3MXJmC2DELWChR0dJx5KoxUrFel

@startuml
actor "Research Participant" as rp
participant "Participant Portal" as pp
participant "IdP\n(Keycloak)" as idp
participant "Key Relay Service" as krs
participant "Relay Policy Agent\n(OPA)" as opa
participant "Relay Keyfile\n(Keycloak?)" as kf
participant "Consents Service" as cs


== Authentication ==

rp -> pp: Submits login info: \nusername, pass
pp -> idp: Authenticates w/ participant's login info

alt Successful authentication
  idp --> pp: Authentication token
  pp --> rp: "Login successful" \n"You will soon be redirected to your Consents home"
else Failed to authenticate
  idp --> pp: Authentication failed
  pp --> rp: "Login unsuccessful" \n"Please verfiy and resubmit your credentials"
end


== Get Consents ==

pp -> krs: GET /consents \nwith Auth token
note right
  GET both default and project
  consents in once request,
  to save time
end note

krs -> opa: Requests participant study identifier
opa -> opa: Perform authorization on: \nParticipant Portal (via api key), \nparticipant (via auth token)

alt Authorized to retrieve this participant's identifier
  opa -> kf: Fetch participant \nassociated with auth token
  kf --> opa: {study_identifier}
  opa --> krs: {study_identifier}

  krs -> cs: GET /participants/{study_identifier}/default_consent
  cs --> krs: 200 OK \nBody: default consent

  krs -> cs: GET /participants/{study_identifier}/project_consents
  cs --> krs: 200 OK \nBody: [project consents]

  krs --> pp: 200 OK \nBody: default consent, [project consents]

  pp -> pp: Caches consents data
  pp ->> rp: Displays Consents home page

else Not authorized to retrieve this participant's identifier
  opa --> krs: Not authorized to retrieve this study identifier
  krs --> rp: 401 Unauthorized
end


== Modify Default Consent ==

rp -> pp: Submits Default Consent modification

pp -> krs: PUT /default_consents \nwith Auth token

krs -> opa: Requests participant study identifier
opa -> opa: Perform authorization on: \nParticipant Portal (via api key), \nparticipant (via auth token)

alt Authorized to retrieve this participant's identifier
  opa -> kf: Fetch participant \nassociated with auth token
  kf --> opa: {study_identifier}
  opa --> krs: {study_identifier}

  krs -> cs: PUT /participants/{study_identifier}/default_consents
  cs --> krs: default consent

  krs --> pp: default consent

  pp -> pp: Updates Consents cache
  pp --> rp: Reloads Consents home page

else Not authorized to retrieve this participant's identifier
  opa --> krs: Not authorized to retrieve this study identifier
  krs --> rp: 401 Unauthorized
end


== Modify Project Consent ==

rp -> pp: Submits Project Consent modification

pp -> krs: PUT /project_consents \nwith Auth token \nBody: {project_application_id}


krs -> opa: Requests participant study identifier
opa -> opa: Perform authorization on: \nParticipant Portal (via api key), \nparticipant (via auth token)

alt Authorized to retrieve this participant's identifier
  opa -> kf: Fetch participant \nassociated with auth token
  kf --> opa: {study_identifier}
  opa --> krs: {study_identifier}

  krs -> cs: PUT /participants/{study_identifier}/project_consents
  cs --> krs: 200 OK \nBody: project consent

  krs --> pp: 200 OK \nBody: project consent

  pp -> pp: Updates Consents cache
  pp --> rp: Reloads Consents home page

else Not authorized to retrieve this participant's identifier
  opa --> krs: Not authorized to retrieve this study identifier
  krs --> rp: 401 Unauthorized
end


== Session End (due to timeout or participant quit/logout) ==

pp -> pp: Clear cache and token \nfor this participant
@enduml