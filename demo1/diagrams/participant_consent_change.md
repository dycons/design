For use in:
http://www.plantuml.com/plantuml/uml/xLLDRzms4BtpLmpsKXjmi6xHqm9ku-OZAD8FmOuFHLq4Osgqoov5CXpAXXhalvUYAArMsMpX0aLHD5VDcpTlPlX6JpcW3qsj1SfWFIokY0czt421FYYf79gm06JmJhZT9rXCub1O7r33DVbx9UdMd5JUca2cF4sfRfRwVLb4p6jgfRQuxJ6gd86kI6CxKFOAMCX8wwqUeeNLIhPmkIOJRasMYO9CKA4TGB5EfJHDAdxRbznM-nMVMSCnbVUAIXP2N5p0PHCs7Q_4eAo1YmiXl8Cdtu1pAxXkxceL6BHTAmFAL7O5jwPXyWPhEWE7pC8bUEnpDIKZXeSbJ7HyCQKH0dM8z58ISzLem3qP0Y9VfCqoPXg3tL87SJt2HyJYXqJD8-42ujn-iGqyAAs1RKox8_1KAayoK1b9eBMDXt4u6rlJGf1cWfUeT89CPT5nKLNAEQYgCJDTXIPaWdlobMe1JGcUE8swroIZpeuQDKT5fkmtzOh2JctSKnwyvnMyUl46bZB7OeK75JP9O9wLiO70g_Kc24ZWEnjZ9LNOw904E6z_9zc5Hnfbm1f9uEbTGnpEOYnOOBmd2AgcJXXqn49uXjGnhk2g1_Dqyy2XALjGgQTAaHSHEcOKv2lhwpHgwzKV_JYjwLxQ0I-Ut2i4T0gsr9wUHSoaK0wEVP_sp-moC_ShzHIyegw9ZUBP0vr81CWYjzKANbAGcoasraLcAnM6o9h6lIih82R1aw71zwdzjplk3vay8VWW82BweSfXkvFYlFmuOvcN-JQlhriaZnM-EZ-7drz7qT_PibsDYn-mZw-MdyjGZO-M-pMZOK3_DbJCLZekx-mJ16uyKyzGReZ7A9GOS03qJdoks6biUT_ku71DelVyJpO0_ls7abl_Ivgv5SP199bVdty9DsQNlpF_ZxPKLGlFysXo6vyurtDKtINdGpKz6yND6nWVpf7pyTdW_wp1Yvl76tnckPbdPWQRH_VSS-DA33InXkpijFydko9jiVpFswV8T-IuVUQee_Pn2NpCFkDTUpzWqJcT-UAAFuZF1lkN3VRuVre67_RRSV3_q7xNnAoiWHUcX9EoeOuvg9fi4y3wgJfurwYmr7OTGwV9dvFVl2Rq_NW0JJcwgxB-eox5qrYygVMV

@startuml
actor "Research Participant" as rp
participant "Participant Portal Service\n(frontend)" as pp
participant "IdP\n(Keycloak)" as idp
participant "Relay Service" as rs
control "Relay Policy Agent\n" as rpa
entity "Relay Keyfile\n(Keycloak?)" as kf
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

pp -> rs: GET /consents \nwith Auth token
note right
  GET both default and project
  consents in once request,
  to save time
end note

rs -> rpa: Requests participant study identifier
rpa -> rpa: Perform authorization on: \nParticipant Portal (via api key), \nparticipant (via auth token)

alt Authorized to retrieve this participant's identifier
  rpa -> kf: Fetch participant \nassociated with auth token
  kf --> rpa: {study_identifier}
  rpa --> rs: {study_identifier}

  rs -> cs: GET /participants/{study_identifier}/default_consent
  cs --> rs: 200 OK \nBody: default consent

  rs -> cs: GET /participants/{study_identifier}/project_consents
  cs --> rs: 200 OK \nBody: [project consents]

  rs --> pp: 200 OK \nBody: default consent, [project consents]

  pp -> pp: Caches consents data
  pp ->> rp: Displays Consents home page

else Not authorized to retrieve this participant's identifier
  rpa --> rs: Not authorized to retrieve this study identifier
  rs --> rp: 401 Unauthorized
end


== Modify Default Consent ==

rp -> pp: Submits Default Consent modification

pp -> rs: PUT /default_consents \nwith Auth token

rs -> rpa: Requests participant study identifier
rpa -> rpa: Perform authorization on: \nParticipant Portal (via api key), \nparticipant (via auth token)

alt Authorized to retrieve this participant's identifier
  rpa -> kf: Fetch participant \nassociated with auth token
  kf --> rpa: {study_identifier}
  rpa --> rs: {study_identifier}

  rs -> cs: PUT /participants/{study_identifier}/default_consents
  cs --> rs: default consent

  rs --> pp: default consent

  pp -> pp: Updates Consents cache
  pp --> rp: Reloads Consents home page

else Not authorized to retrieve this participant's identifier
  rpa --> rs: Not authorized to retrieve this study identifier
  rs --> rp: 401 Unauthorized
end


== Modify Project Consent ==

rp -> pp: Submits Project Consent modification

pp -> rs: PUT /project_consents \nwith Auth token \nBody: {project_application_id}


rs -> rpa: Requests participant study identifier
rpa -> rpa: Perform authorization on: \nParticipant Portal (via api key), \nparticipant (via auth token)

alt Authorized to retrieve this participant's identifier
  rpa -> kf: Fetch participant \nassociated with auth token
  kf --> rpa: {study_identifier}
  rpa --> rs: {study_identifier}

  rs -> cs: PUT /participants/{study_identifier}/project_consents
  cs --> rs: 200 OK \nBody: project consent

  rs --> pp: 200 OK \nBody: project consent

  pp -> pp: Updates Consents cache
  pp --> rp: Reloads Consents home page

else Not authorized to retrieve this participant's identifier
  rpa --> rs: Not authorized to retrieve this study identifier
  rs --> rp: 401 Unauthorized
end


== Session End (due to timeout or participant quit/logout) ==

pp -> pp: Clear cache and token \nfor this participant
@enduml