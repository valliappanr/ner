## Entity Persister

Once the named entities are extracted from the document, the final step is to persist the document. Any No-SQL be used to store it. 

Have explored the option to use Click-House database, as this simulates the columnar database and can run quickly in a single node.



Sample code to process the data is as below,

```java
import com.ecwid.clickhouse.ClickHouseResponse;
import com.ecwid.clickhouse.mapped.ClickHouseMappedClient;
import com.ecwid.clickhouse.raw.RawValues;
import com.ecwid.clickhouse.transport.HttpTransport;
import com.ecwid.clickhouse.transport.httpclient.ApacheHttpClientTransport;
import persist.model.LocDocMapping;
import persist.model.MissedEntityMapping;
import persist.model.OrgDocMapping;
import persist.model.UserDocMapping;

import java.util.List;
import java.util.stream.Collectors;


public class ClickHouseClient {

    public static final String ENTITIES_USER_DOC = "entities.user_doc";
    public static final String ENTITIES_ORG_DOC = "entities.org_doc";
    public static final String ENTITIES_LOC_DOC = "entities.loc_doc";

    private String dbUrl;
    private HttpTransport httpTransport;
    private ClickHouseMappedClient client;

    public ClickHouseClient(String dbUrl) {
        this.dbUrl = dbUrl;
        this.httpTransport = new ApacheHttpClientTransport();
        this.client = new ClickHouseMappedClient(httpTransport);
    }

    public void writeRecord(String docId, String link, List<String> userNames, List<String> organisations, long timestamp) {
        List<RawValues> userDocMappings = createUserDocMappings(docId, link, userNames, timestamp);

        client.getRawClient().insert(dbUrl,
                ENTITIES_USER_DOC, userDocMappings);

        List<RawValues> orgDocMappings = createOrgDocMappings(docId, link, organisations, timestamp);
        client.getRawClient().insert(dbUrl,
                ENTITIES_ORG_DOC, orgDocMappings);

        List<RawValues> locDocMappings = createLocDocMappings(docId, link,  locations, timestamp);
        client.getRawClient().insert(dbUrl,
                ENTITIES_LOC_DOC, locDocMappings);

    }

    private List<RawValues> createUserDocMappings(String docId, String link, List<String> userNames, long timestamp) {
        return userNames.stream().map(userName -> {
            return new UserDocMapping(docId, userName, link, timestamp).convertToRawValues();
        }).collect(Collectors.toList());
    }

    private List<RawValues> createOrgDocMappings(String docId, String link, List<String> organisations, long timestamp) {
        return organisations.stream().map(org -> {
            return new OrgDocMapping(docId, org, link, timestamp).convertToRawValues();
        }).collect(Collectors.toList());
    }

    private List<RawValues> createLocDocMappings(String docId, String link, List<String> locations, long timestamp) {
        return locations.stream().map(loc -> {
            return new LocDocMapping(docId, loc, link, timestamp).convertToRawValues();
        }).collect(Collectors.toList());
    }

}
```



Code to consume the Kafka message and persist to the database.

```java
   void consumeRecord(String docId, String jsonPayload) throws IOException {
        ObjectMapper objectMapper = new ObjectMapper();
        EntityPersisterRequest entityPersisterRequest = objectMapper.readValue(jsonPayload,
                EntityPersisterRequest.class);
        LOGGER.debug("Persisting Request {}", entityPersisterRequest);
        EntityExtrationResponse entityExtrationResponse = entityPersisterRequest.getEntityExtrationResponse();
        Document document = entityPersisterRequest.getDocument();
        String linkUrl = document.getLinkUrl();
        long creationTime =document.getCreationTime().getTime();
        clickHouseClient.writeRecord(docId, linkUrl, entityExtrationResponse.getPeople(),
                entityExtrationResponse.getOrganisations(), entityExtrationResponse.getLocations(),creationTime);
        LOGGER.debug("Successfully persisted entity {}", entityPersisterRequest);
    }
```

## Entity Report

Entity report is a spring boot app, which reads the data from the No-SQL database and present it to the user on the recognized entities.