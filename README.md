# Repro for issue 9395

## Versions

firebase-tools: v14.22.0<br>

## Steps to reproduce

1. Delete all indexes you have in Firestore (default) database
2. Run `firebase deploy --only firestore:indexes --project PROJECT_ID`
   - Initial indexes

```json
{
  "indexes": [
    {
      "collectionGroup": "myVectorCollection",
      "queryScope": "COLLECTION",
      "fields": [
        {
          "fieldPath": "embedding",
          "vectorConfig": {
            "dimension": 768,
            "flat": {}
          }
        }
      ],
      "density": "SPARSE_ALL"
    }
  ],
  "fieldOverrides": []
}
```

3. Wait for the indexes to build(~2-5 minutes)
4. Run `firebase deploy --only firestore:indexes --project PROJECT_ID` again

```
$ firebase deploy --only firestore:indexes --project PROJECT_ID

=== Deploying to 'PROJECT_ID'...

i  deploying firestore
i  firestore: ensuring required API firestore.googleapis.com is enabled...
i  firestore: ensuring required API firestore.googleapis.com is enabled...
i  firestore: reading indexes from firestore.indexes.json...
i  cloud.firestore: checking firestore.rules for compilation errors...
✔  cloud.firestore: rules file firestore.rules compiled successfully
i  firestore: deploying indexes...
i  firestore: The following indexes are defined in your project but are not present in your firestore indexes file:
        (myVectorCollection) -- (embedding,VECTOR<768>)  -- Density:SPARSE_ALL
? Would you like to delete these indexes? Selecting no will continue the rest of the deployment. (y/N)
```

Prompt shouldn't appear since there are no changes in the indexes. Though, answering with `No` will raise an error

```
✔ Would you like to delete these indexes? Selecting no will continue the rest of the deployment. No

Error: Request to https://firestore.googleapis.com/v1/projects/PROJECT_ID/databases/(default)/collectionGroups/myVectorCollection/indexes had HTTP Error: 400, No valid order or array config provided: field_path:          "__name__"
```

5. Pull the latest Firestore indexes. Run `firebase firestore:indexes --project PROJECT_ID`
   - Outputs

```json
{
  "indexes": [
    {
      "collectionGroup": "myVectorCollection",
      "queryScope": "COLLECTION",
      "fields": [
        {
          "fieldPath": "__name__",
          "order": "ASCENDING"
        },
        {
          "fieldPath": "embedding",
          "vectorConfig": {
            "dimension": 768,
            "flat": {}
          }
        }
      ],
      "density": "SPARSE_ALL"
    }
  ],
  "fieldOverrides": []
}
```

6. Update the local `firestore.indexes.json` with the output from above
7. Run `firebase deploy --only firestore:indexes --project PROJECT_ID` again

```
$ firebase deploy --only firestore:indexes --project PROJECT_ID

=== Deploying to 'PROJECT_ID'...

i  deploying firestore
i  firestore: ensuring required API firestore.googleapis.com is enabled...
i  firestore: ensuring required API firestore.googleapis.com is enabled...
i  firestore: reading indexes from firestore.indexes.json...
i  cloud.firestore: checking firestore.rules for compilation errors...
✔  cloud.firestore: rules file firestore.rules compiled successfully
i  firestore: deploying indexes...
i  firestore: The following indexes are defined in your project but are not present in your firestore indexes file:
        (myVectorCollection) -- (embedding,VECTOR<768>)  -- Density:SPARSE_ALL
? Would you like to delete these indexes? Selecting no will continue the rest of the deployment. (y/N)
```

Prompt shouldn't appear since there are no changes in the indexes. Though, answering with `No` will raise an error

```
✔ Would you like to delete these indexes? Selecting no will continue the rest of the deployment. No

Error: Request to https://firestore.googleapis.com/v1/projects/PROJECT_ID/databases/(default)/collectionGroups/myVectorCollection/indexes had HTTP Error: 400, No valid order or array config provided: field_path:          "__name__"
```

## Notes

The request seems to be malformed(?). Checking the debug logs there are 2 instances of `fieldPath` `__name__`, one of which does not have an `order` value

```
Creating new index: {"collectionGroup":"myVectorCollection","queryScope":"COLLECTION","density":"SPARSE_ALL","fields":[{"fieldPath":"__name__","order":"ASCENDING"},{"fieldPath":"embedding","vectorConfig":{"dimension":768,"flat":{}}},{"fieldPath":"__name__"}]}
```

## Post Fix

Using firebase-tools v14.23.0

1. Run `firebase deploy --only firestore:indexes --project PROJECT_ID`
```
$ firebase --version
14.23.0
$ firebase deploy --only firestore:indexes --project PROJECT_ID

=== Deploying to 'PROJECT_ID'...

i  deploying firestore
i  firestore: ensuring required API firestore.googleapis.com is enabled...
i  firestore: ensuring required API firestore.googleapis.com is enabled...
i  firestore: reading indexes from firestore.indexes.json...
i  cloud.firestore: checking firestore.rules for compilation errors...
✔  cloud.firestore: rules file firestore.rules compiled successfully
i  firestore: deploying indexes...
✔  firestore: deployed indexes in firestore.indexes.json successfully for (default) database

✔  Deploy complete!
```