# Hyperledger Aries Credential Verification
## Setup

Install docker desktop for either Mac, Windows or Linux https://docs.docker.com/desktop/

### Faber
Open a terminal window for Faber
```
git clone https://github.com/openwallet-foundation/acapy
cd acapy/demo
LEDGER_URL=http://test.bcovrin.vonx.io ./run_demo faber --events --no-auto --bg
```
After the docker has started, use the following command to show the output of the logs
```
docker logs -f faber
```
Open a browser tab and confirm you can access the Swagger APIs for Faber at http://localhost:8021

### Alice
Open another terminal window for Alice
```
LEDGER_URL=http://test.bcovrin.vonx.io ./run_demo alice --events --no-auto --bg
```
After the docker has started, use the following command to show the output of the logs
```
docker logs -f alice
```
Open a browser tab and confirm you can access the Swagger APIs for Alice at http://localhost:8031.

## Creating a connection
Looking at the log output from Faber, copy the entire block of the invitation object, from the curly brackets {}, excluding the trailing comma.

For example
```
{"@type": "https://didcomm.org/out-of-band/1.1/invitation", "@id": "5b5bae45-5503-45fd-8981-9441869d5365", "label": "faber.agent", "handshake_protocols": ["https://didcomm.org/didexchange/1.1"], "services": [{"id": "#inline", "type": "did-communication", "recipientKeys": ["did:key:z6Mkra7BDXf6SCp8Lem8sKvqURG3ZL2p4B1oLLi1DBL2zABn#z6Mkra7BDXf6SCp8Lem8sKvqURG3ZL2p4B1oLLi1DBL2zABn"], "serviceEndpoint": "http://192.168.65.9:8020"}]}
```

To receive the invitation by Alice, go the Alice's API http://localhost:8031 and find the ```/out-of-band/receive-invitation``` endpoint.

Replace the pre-populated text with the invitation object from Faber.

Next, we need Alice to accept the invitation.  Looking at the log out for Alice, we can see the ```contection_id```.  

In Alice's API, find the ```/didexchange/{conn_id}/accept-invitation``` endpoint, and use the connection_id found int Alice's log output.

If you look at the log output for Faber, you should also see the connection ```state``` has changed to ```active```.  Please note that the connection id will be different betwee Alice and Faber, but has the same ```invitation_msg_id```.

Check the list of connections for both Alice and Faber by executing /connections endpoint.


### Preparing a credential
Find the corresponding schema using the Faber API ```/schemas/created``` endpoint. Copy this value into a text editor for later use.

Next, find the credential definition using the Faber API ```/credential-definitions/created``` endpoint. Copy this value into a text editor for later use.

### Issuing a credential
Now we are ready to issue a credential. 

Relace the following values for the curl command below and execute it in a new terminal window. 

- ```connection_id``` with the connection ID in Faber log output
- ```schema_id``` the Id of the schema Faber created (use ```GET /schemas/created```) and,
- ```schema_issuer_did``` the Faber public DID (use ```GET /wallet/DID/public```)
- ```schema_version``` with the new schema version from the Faber log output
- ```cred_def_id``` the Id of the credential definition Faber created (use ```GET /credential-definitions/created```)
- ```issuer_did``` the Faber public DID (use ```GET /wallet/DID/public```),

Submit the Faber API ```POST /issue-credential-v2.0/send-offer``` using this example as a template:
```
{
  "connection_id": "30385f20-30f9-4949-ab94-e1ef2b09740f",
  "filter": {
    "indy": {
      "schema_id": "BdH67XxU3cGJAtnkf6pZpS:2:degree schema:76.46.99",
      "schema_issuer_did": "BdH67XxU3cGJAtnkf6pZpS",
      "schema_name": "degree schema",
      "schema_version": "76.46.99",
      "cred_def_id": "BdH67XxU3cGJAtnkf6pZpS:3:CL:2706685:faber.agent.degree_schema",
      "issuer_did": "BdH67XxU3cGJAtnkf6pZpS"
    }
  },
  "comment": "Issuing degree credential",
  "credential_preview": {
    "@type": "https://didcomm.org/issue-credential/2.0/credential-preview",
    "attributes": [
      {
        "name": "degree",
        "value": "Bachelor of Computer Science"
      },
      {
        "name": "birthdate_dateint",
        "value": "19900101"
      },
      {
        "name": "date",
        "value": "2024-03-19"
      },
      {
        "name": "timestamp",
        "value": "1710864000"
      },
      {
        "name": "name",
        "value": "John Doe"
      }
    ]
  },
  "auto_remove": true,
  "trace": true
}
```

Look at the log outpus for both Faber and Alice and you will notice a number of communication exchanges to accept the credential.

To save the credential in Alice's wallet, copy the ```cred_ex_id``` from Alice's credential log output and then using Alice's API, execute the ```/issue-credential-2.0/records/{cred_ex_id}/store``` endpoint with the ```cred_ex_id```.  Ignore the payload field for now.

