<script setup>

import { useCollection } from 'vuefire'
import { addDoc, collection, updateDoc, where } from 'firebase/firestore'
import { doc, deleteDoc } from 'firebase/firestore'
import { useFirestore } from 'vuefire';
import { ref } from "vue";
import { getAuth,signOut } from "firebase/auth";
import { getStorage,ref as refStorage, uploadBytes, getDownloadURL} from "firebase/storage";
import { query } from 'firebase/database';
import { useRouter } from 'vue-router';
const auth = getAuth();
const router = useRouter();
let db = useFirestore();
//const auth = getAuth();
let usuario = getAuth().currentUser

let col = collection(db, 'todos');
let que = query(col,where("usuario", "==", usuario.uid));
let todos = useCollection(que);

//let list = useCollection(query(colleccion ,where("idusuario","==",id)));
let file=null;

let contenidoNota="";

console.log(usuario);

  //funcion crear nota
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
      prioridad:"Baja",
      usuario:usuario.uid
    }).then((e)=>console.log(e));
    //
  })
}

  //funcion para eliminar nota 
function eliminarNota(id){
  deleteDoc(doc(db,"todos",id))
}


//funciones para cambiar la prioridad
function cambiarPrioridad(id){
  const notaActualizada = doc(db,"todos",id)
  updateDoc(notaActualizada,{
    prioridad:"Alta"
  });
}

function cambiarPrioridadM(id){
  const notaActualizada = doc(db,"todos",id)
  updateDoc(notaActualizada,{
    prioridad:"Media"
  });
}

//funcion para subir fichero
function subirAjdunto(e){
  file = e.target.files[0];
  console.log(file);
}

  //funcion para cerrarsesion
function cerrarSesion()
{
    signOut(auth).then(() => {
        // Sign-out successful.
          usuario.value="";
         router.push({ path: '/' })
    }).catch((error) => {
        // An error happened.
    });

}
</script>


<template>
  <header>
    <input type="file" name="file" id="" @change="subirAjdunto($event)">
    <input type="text" name="" id="" v-model="contenidoNota"><button @click="altaNota">Alta de nota</button>
    <button v-on:click="cerrarSesion()">Cerrar Sesion</button>
    <div class="wrapper">

    <ul>
      <p>Notas de {{ usuario.displayName }} {{  }}</p>
      <p>Notas Totales: {{ todos.length }}</p>
        <li v-for="todo in todos" :key="todo.id">
          
           <span>Texto:{{ todo.texto}}, Prioridad:{{ todo.prioridad }}, User:{{ usuario.uid }} </span> 
           <button @click="eliminarNota(todo.id)">Eliminar</button>
           <button @click="cambiarPrioridad(todo.id)">Prioridad Alta</button>
           <button @click="cambiarPrioridadM(todo.id)">Prioridad Media</button>
        </li>
    </ul>
    
    
    </div>

  </header>
  </template>


<style>
#app {
  font-family: 'Arial', sans-serif;
  text-align: center;
}

h1 {
  color: #333;
}

input {
  padding: 10px;
  margin: 10px 0;
}

ul {
  list-style-type: none;
  padding: 0;
}

li {
  background-color: #f0f0f0;
  margin: 5px 0;
  padding: 10px;
  border-radius: 5px;
  display: flex;
  justify-content: space-between;
  align-items: center;
  color: black;
}

button {
  background-color: #dc3545;
  color: #fff;
  border: none;
  padding: 5px 10px;
  cursor: pointer;
  border-radius: 3px;
}
</style>


