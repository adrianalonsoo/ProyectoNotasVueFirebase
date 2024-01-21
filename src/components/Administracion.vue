<script setup>



import { getAuth, signInWithEmailAndPassword } from "firebase/auth";



import { useCollection } from 'vuefire'
import { addDoc, collection, updateDoc, where } from 'firebase/firestore'
import { doc, deleteDoc } from 'firebase/firestore'
import { useFirestore } from 'vuefire';
import { ref } from "vue";
import { getStorage,ref as refStorage, uploadBytes, getDownloadURL} from "firebase/storage";
import { query } from 'firebase/database';
let db = useFirestore();
//const auth = getAuth();
const auth = getAuth();

let todos = useCollection(collection(db,"todos"));

let usuario = ref(auth.currentUser);

//let list = useCollection(query(colleccion ,where("idusuario","==",id)));
let file=null;

let contenidoNota="";

const email="admin@admin.com";
const password="administrador";


function altaNota(){
  

    const storage = getStorage();
    const storageRef = refStorage(storage, "hola");
    uploadBytes(storageRef, file).then((snapshot) => {
    console.log('Uploaded a blob or file!');
    return getDownloadURL(snapshot.ref)
  }).then(downloadURL=>{
    console.log("Download URL",downloadURL);
    console.log(contenidoNota);
    const docRef = addDoc(collection(db,"todos"),{
      texto:contenidoNota,
      prioridad:"baja",
      
    }).then((e)=>console.log(e));
    //
  })
}

function eliminarNota(id){
  deleteDoc(doc(db,"todos",id))
}



function cambiarPrioridad(id){
  const notaActualizada = doc(db,"todos",id)
  updateDoc(notaActualizada,{
    prioridad:"alta"
  });
}


function inicio(){
    signInWithEmailAndPassword(auth, email, password)
  .then((userCredential) => {
    // Signed in 
    const usuario = userCredential.user;
    // ...
  })
  .catch((error) => {
    const errorCode = error.code;
    const errorMessage = error.message;
  });

}


function cambiarPrioridadM(id){
  const notaActualizada = doc(db,"todos",id)
  updateDoc(notaActualizada,{
    prioridad:"Media"
  });
}

</script>


<template>
    <header>

    <input type="text" name="" id="" v-model="contenidoNota"><button @click="altaNota">Alta de nota</button>
    <div class="wrapper">

    <ul>
      <p>Notas de: {{ email }}</p>
      <p>Notas Totales: {{ todos.length }}</p>
        <li v-for="todo in todos" :key="todo.id">
          
           <span>{{ todo.texto}} {{ todo.prioridad }} </span> 
           <button @click="eliminarNota(todo.id)">Eliminar</button>
           <button @click="cambiarPrioridad(todo.id)">Prioridad Alta</button>
           <button @click="cambiarPrioridadM(todo.id)">Prioridad Media</button>
        </li>
    </ul>
    
    
    </div>

  </header>
</template>