<template>
  <div>
    <p>Data added: {{ addedData }}</p>
    <p>Data updated: {{ updatedData }}</p>
    <p>Data deleted: {{ deletedData }}</p>
    <button @click="addData">Add Data</button>
    <button @click="readData">Read Data</button>
    <button @click="updateData">Update Data</button>
    <button @click="deleteData">Delete Data</button>
    <br>
    <h4>Upload Image</h4>
    <p>File Added: {{ uploadedFile }}</p>
    <p>File Deleted: {{ deletedFile }}</p>
    Click Here to upload IMAGE <input type="file" id="uploadFile" accept="image/png, image/jpeg" />
    <button @click="submitImage">Submit</button>
    <br>
    <button @click="readImage">See Image</button>
    <button @click="deleteImage">Delete Image</button>
    <h2>Customer List</h2>
    <ul>
      <li v-for="customer in customers" :key="customer.ssn">
          {{ customer.name }} - {{ customer.email }}
      </li>
    </ul>
    <h2>File List</h2>
    <ul>
      <li v-for="file in files" :key="file.idx">
          <img :src="file.image" :alt="file.idx" :height="300" :width="auto">
      </li>
    </ul>
  </div>
</template>

<script setup>
  import { ref, onMounted } from 'vue';
  let a = 0, b = 0, c = 0;
  const dbName = "the_name";
  const imageId = "IMGFILE-";
  const custId = "Customer-";
  
  const addedData = ref(null);
  const updatedData = ref(null);
  const deletedData = ref(null);
  const uploadedFile = ref(null);
  const deletedFile = ref(null);
  let customers = ref([]);
  let files = ref([]);

  
  const addData = () => {
    const request = indexedDB.open(dbName, 2);
    const customerToAdd = { ssn: custId+(b++), name: "Alice in Wonderland", age: 28, email: "alice@example.com" };

    request.onerror = (event) => {
      console.error("Error opening database:", event.target.error);
    };

    request.onupgradeneeded = (event) => {
      const db = event.target.result;

      const objectStore = db.createObjectStore("customers", { keyPath: "ssn" });
      objectStore.createIndex("name", "name", { unique: false });
      objectStore.createIndex("email", "email", { unique: false });
    };
  
    request.onsuccess = (event) => {
      const db = event.target.result;
  
      const addTransaction = db.transaction("customers", "readwrite");
      const customerObjectStore = addTransaction.objectStore("customers");
  
      const addRequest = customerObjectStore.add(customerToAdd);
  
      addRequest.onsuccess = (event) => {
          console.log("Data added successfully");
          addedData.value = "Yes";
      };
  
      addRequest.onerror = (event) => {
          console.error("Error adding data", event.target.error);
          addedData.value = "No";
      };
  
      addTransaction.oncomplete = () => {
          console.log("Add transaction completed");
          db.close();
      };
    };
  };

  const resetFile = () => {
    let file= document.getElementById('uploadFile');
    file.value = '';
  }

  const submitImage = () => {
      const request = indexedDB.open("file_img", 2);
  
      request.onerror = (event) => {
      console.error("Error opening database image:", event.target.error);
      };
  
      request.onupgradeneeded = (event) => {
      const db = event.target.result;
  
      const objectStore = db.createObjectStore("files", { keyPath: "idx" });
      objectStore.createIndex("image", "image", { unique: false });
      };

      let fileImage = document.getElementById('uploadFile').files[0];
      const blobData = new Blob([fileImage], {type:fileImage.type});
      const objFile = {idx: imageId+(a++), image: blobData};
  
      request.onsuccess = (event) => {
        const db = event.target.result;
    
        const addTransaction = db.transaction("files", "readwrite");
        const fileObjStore = addTransaction.objectStore("files");
    
        const addRequest = fileObjStore.add(objFile);
    
        addRequest.onsuccess = (event) => {
            console.log("File added successfully");
            uploadedFile.value = "Yes";
        };
    
        addRequest.onerror = (event) => {
            console.error("Error adding file", event.target.error);
            uploadedFile.value = "No";
        };
    
        addTransaction.oncomplete = () => {
            resetFile();
            console.log("Add File transaction completed");
            db.close();
        };
      };
      
  };

  const readData = () => {
    const request = indexedDB.open(dbName, 2);

    request.onerror = (event) => {
      console.error("Error opening database:", event.target.error);
    };

    request.onsuccess = (event) => {
      const db = event.target.result;
      customers.value = [];

      const readTransaction = db.transaction("customers", "readonly");
      const customerObjectStore = readTransaction.objectStore("customers");

      const customersCursor = customerObjectStore.openCursor();

      customersCursor.onsuccess = (event) => {
        const cursor = event.target.result;
        if (cursor) {
          customers.value.push(cursor.value);
          cursor.continue();
        } else {
          console.log("Data read successfully");
          db.close();
        }
      };

      console.log(customers);

      customersCursor.onerror = (event) => {
        console.error("Error reading data", event.target.error);
        db.close();
      };
    };
  };

  const readImage = () => {
    const request = indexedDB.open("file_img", 2);

    request.onerror = (event) => {
      console.error("Error opening database image:", event.target.error);
    };

    request.onsuccess = (event) => {
      const db = event.target.result;
        files.value = [];

      const readTransaction = db.transaction("files", "readonly");
      const fileObjStore = readTransaction.objectStore("files");

      const fileCursor = fileObjStore.openCursor();
      

      fileCursor.onsuccess = (event) => {
        const cursor = event.target.result;

        if (cursor) {
          const blobImg = cursor.value.image;
          console.log(blobImg);
          console.log(cursor.value.idx);
          var imageSrc = URL.createObjectURL(blobImg);
          const fileObj = {idx:cursor.value.idx, image:imageSrc}
          console.log(fileObj);
          files.value.push(fileObj);
          cursor.continue();
        } else {
          console.log("Data read successfully");
          db.close();
        }
      };

      fileCursor.onerror = (event) => {
        console.error("Error reading data", event.target.error);
        db.close();
      };
    };
  };

  const updateData = () => {
    const request = indexedDB.open(dbName, 2);
    
    const updatedCustomerData = { ssn: custId+(c++), name: "Alice Updated", age: 30, email: "alice.updated@example.com" };

    request.onerror = (event) => {
      console.error("Error opening database:", event.target.error);
    };

    request.onsuccess = (event) => {
      const db = event.target.result;

      const updateTransaction = db.transaction("customers", "readwrite");
      const customerObjectStore = updateTransaction.objectStore("customers");

      const updateRequest = customerObjectStore.put(updatedCustomerData);

      updateRequest.onsuccess = (event) => {
        console.log("Data updated successfully");
        updatedData.value = "Yes";
      };

      updateRequest.onerror = (event) => {
        console.error("Error updating data", event.target.error);
        updatedData.value = "No";
      };

      updateTransaction.oncomplete = () => {
        console.log("Update transaction completed");
        db.close();
      };
    };
  };

  const deleteData = () => {
    const request = indexedDB.open(dbName, 2);

    request.onerror = (event) => {
      console.error("Error opening database:", event.target.error);
    };

    request.onsuccess = (event) => {
      const db = event.target.result;

      const deleteTransaction = db.transaction("customers", "readwrite");
      const deleteObjectStore = deleteTransaction.objectStore("customers");
      const keyToDelete = custId+(--b);
      c--;
      console.log(keyToDelete);
      const deleteRequest = deleteObjectStore.delete(keyToDelete);

      deleteRequest.onsuccess = (event) => {
        console.log("Data deleted successfully");
        deletedData.value = "Yes";
      };

      deleteRequest.onerror = (event) => {
        console.error("Error deleting data", event.target.error);
        deletedData.value = "No";
      };

      deleteTransaction.oncomplete = () => {
        console.log("Delete transaction completed");
        db.close();
      };
    };
  };

  const deleteImage = () => {
    const request = indexedDB.open("file_img", 2);

    request.onerror = (event) => {
      console.error("Error opening database image:", event.target.error);
    };

    request.onsuccess = (event) => {
      const db = event.target.result;

      const deleteTransaction = db.transaction("files", "readwrite");
      const deleteObjectStore = deleteTransaction.objectStore("files");
      const keyToDelete = imageId+(--a);
      console.log(keyToDelete);
      const deleteRequest = deleteObjectStore.delete(keyToDelete);

      deleteRequest.onsuccess = (event) => {
        console.log("Image deleted successfully");
        deletedFile.value = "Yes";
      };

      deleteRequest.onerror = (event) => {
        console.error("Error deleting image", event.target.error);
        deletedFile.value = "No";
      };

      deleteTransaction.oncomplete = () => {
        console.log("Delete image transaction completed");
        db.close();
      };
    };
  };

</script>