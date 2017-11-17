# Firestore Codelab

Following the steps of [this codelab](https://codelabs.developers.google.com/codelabs/firestore-android/index.html) to get a sense of the new Cloud Firestore product in Firebase. You can see final example code [here](https://github.com/firebase/quickstart-android/tree/master/firestore/app/src/main/java/com/google/firebase/example/fireeats)

## Codelab Walkthrough

1. Make sure you have Android Studio installed. Requires v 2.3 or higher. Mine is currently at 3.0

2. Download the starter code using ```git clone https://github.com/firebase/friendlyeats-android``` - and import it into Android Studio as a new project. You might get alerts to update specific packages when you do this (e.g., I had to install Build Tools 26.0.2).

3. Open the imported Android Studio project. Let Gradle do its thing to build the app. You should see this error because Firebase configuration is not done. This will be corrected later in step ().
```
Error:Execution failed for task ':app:processDebugGoogleServices'.
> File google-services.json is missing. The Google Services Plugin cannot function without it. 
   Searched Location: 
  ../friendlyeats-android/app/src/debug/google-services.json
  ../friendlyeats-android/app/google-services.json
```

4. Create a Firebase project (e.g, DataOnFire) and enter Console. In the Database section, check "Try Firestore Beta". Leave defaults and click "Enable". 
    * This sets up "Locked" security rules by default. At this point you have the database console pointing to "Firestore" dashboard - but note that you can ALSO set up a real-time db instance at the same time if you really want to, by using the drop-down at top. 
    * Note the [different tabs](https://console.firebase.google.com/project/data-on-fire/database/firestore/usage) (and how Usage is registered against your Google Cloud account). Note that when you create a Cloud Firestore project, it enables the API in the Cloud API Manager for you.

5. Setup your development environment. First [add Firebase to your Android app](https://firebase.google.com/docs/android/setup). I used the Firebase Assistant on Android Studio (Tools > Firebase) and selected the Analytics product as a default to enable me to configure Firebase. This authenticates AndroidStudio with your Firebase backend (once you sign in) and allows subsequent interactions to be fairly seamless. It will ask you to either create a new project, or connect the Android app to an existing project. I had pre-created one above, so just connected to that. _At this point, it should have created and added the required  google-services.json file for you._

6. Now configure your client app to use the Firestore service. Add the Cloud Firestore Android library to your app/build.gradle file using ```compile 'com.google.firebase:firebase-firestore:11.4.2' ``` if you did step 5 manually. In my case, I found it already existed.

7. Now build and run the app. You should see a log in screen and waiting. This shows the Firebase setup was successful.

8. Next enable EMAIL authentication on Firebase console (Authentication / Sign-in Method) so you can use the default Auth services. We need this in order to establish security rules that are tied to authenticated users.

9. Now update the Firestore [Rules](https://console.firebase.google.com/project/data-on-fire/database/firestore/rules) tab to allow only signed-in users to read/write data using this update rule, and hit PUBLISH.
```
// New rules:
service cloud.firestore {
  match /databases/{database}/documents {
    match /{document=**} {
      allow read, write: if request.auth.uid != null;
    }
  }
}

// Old rules:
service cloud.firestore {
  match /databases/{database}/documents {
    match /{document=**} {
      allow read, write: if false;
    }
  }
}
```
10. Now rebuild and run the app. Enter your email - it should allow you to sign up or sign in according to whether you previously had an account set up or not.

11. [STEP 3] Now write data to Firestore. We have two models (Restaurant and Rating) defined in our src/model directory. We can enter data for these directly into console or programmatically via app.
    * Restaurant and Rating are both "Collections" of data because they are containers for several instances (records) of those types.
    * Each instance (record) is called a "Document". Collections are made up of Documents.
    * A Document can itself contain a reference to another Collection - in this context, that is called a "sub-collection". Note that this is a reference and not a copy - so retrieving Document data does not automatically pull in documents from referenced collections.
    * Here "Restaurants" is a Collection or Restaurant documents. A Restaurant document has a reference to a "Ratings" Collection that stores ratings provided by users for that restaurant.

12. MainActivity - initFirestore to get an instance of the Firestore service.
```
    private void initFirestore() {
        mFirestore = FirebaseFirestore.getInstance();
    }
```

13. Then fill in the helper method to populate data in Firestore (make sure the appropriate imports are resolved) - and removed the placeholder "toast" method invocation. Build and verify by clicking the "Add Random Items" menu option - then check that database adds new items on backend. Each click will add another X random items so watch out. This ends STEP 3.
```
 private void onAddItemsClicked() {
        // Get a reference to the restaurants collection
        CollectionReference restaurants = mFirestore.collection("restaurants");

        for (int i = 0; i < 10; i++) {
            // Get a random Restaurant POJO
            Restaurant restaurant = RestaurantUtil.getRandom(this);

            // Add a new document to the restaurants collection
            restaurants.add(restaurant);
        }
    }
```
14. Notes from STEP 3: 
 * Collections are created implicitly when a document is added
 * add() adds document to Collection with default unique id 
 * Documents can be created using POJOs. In this example, the RestaurantUtil literally creates Restaurant POJOs randomly from fake data, and adds them.

15. [STEP 4] RETRIEVE & DISPLAY DATA FROM FIRESTORE. The first step is to create a Query instance. Here "orderBy" sorts the returned results using the specified attribute, in the specified direction. The "limit" argument specifies how many results I want returned from the matching result set e.g., for efficiency.
```
    private void initFirestore() {
        mFirestore = FirebaseFirestore.getInstance();

        // Get the 50 highest rated restaurants
        mQuery = mFirestore.collection("restaurants")
                .orderBy("avgRating", Query.Direction.DESCENDING)
                .limit(LIMIT);
    }
```

16. We want to bind query results to a RecyclerView so let's set up an Adapter. And have it implement an EventListener that gets called back when there are new "QuerySnapshot" events. These can be of type ADDED (new results added), MODIFIED (existing results changed) and REMOVED (results removed). We handle them with the appropriate onDocumentAdded, onDocumentModified and onDocumentRemoved. Now simply register this listener when you make a Query; that way if the query results (QuerySnapshot) changes over time, you get notified in real time and can adapt your client side UI/UX accordingly. If you do NOT want real-time sync (i.e., no continuous updates, just a 1-time get), just use mQuery.get() instead of mQuery.addSnapshotListener(..)
See code [here](https://github.com/firebase/quickstart-android/blob/master/firestore/app/src/main/java/com/google/firebase/example/fireeats/adapter/FirestoreAdapter.java)

17. 








