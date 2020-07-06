For use in: http://www.plantuml.com/plantuml/uml/ZP9FRzGm4CNl-HHFUmRjKlMK0sefVmxyQSHsRqZLo9wbXiIEF1DWeNntd117ILH43uaKpUyzlpVEXIX5xPgw8ACXObD5rr0yuIKfOQ_ydQBTW0IjsA9Dh3Ek9Q_ONBPjxGof2nvxZj-SOHb8aXNOwTrFl6SbsvjUVcQlVy66bkHLy9A6alq6oYGxiNZw74PALDVx6smj7wchzTQCWi981guYapBclhNpNzb20_IEXJ6giL0dUJgcnDdksUXXySaVFhuhiPsPoVQNQ6TFjyucgJiwZhzpFXhptLU7FYCf9OSNmPvAz1zFy82KP50sbAjrRM8vDtlmk4JdjczUtVmtthQDuGkRsRQeLbIp1V4F9ofm7kiRA81-EJdYsMw7wxTJgz4Oap5w39a-35McAPhn5yXOpqasBO1n5p8fC_oFwxdFKgw55uLyK1n3v-qZEftlpl4XyQzxVXLjFPidMSuuFHVfrJNr7m00

@startuml
actor "Primary Data Steward" as psd
participant "Application Services" as as
participant "Authz Metadata Agent" as ama
participant "Consents Service" as cs

psd -> cs: POST /default_consents
cs -> cs: Create participant linked to these default consents
cs --> psd: 201 Created \nURL: /participants/{study_identifier}/default_consents
psd -> as: POST|PUT /data \nBody: data, {study_identifier}
as -> ama: POST /update_consents/{study_identifier}
ama -> cs: GET /participants/{study_identifier}/project_consents

alt Participant exists in Consents Service
  cs --> ama: 200 OK \nBody: project consents
  ama -> ama: Update consents metadata
  ama --> as: 200 OK
  as -> as: Update data
  as --> psd: 201 Created | 200 OK
else Participant not found in Consents Service
  cs --> ama: 404 Not Found
  ama --> as: 404 Not Found
  as --> psd: 404 Not Found
end
@enduml
