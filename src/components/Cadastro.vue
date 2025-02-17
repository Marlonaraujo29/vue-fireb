<template>
  <div class="max-w-md mx-auto mt-10 p-6 bg-white rounded-lg shadow-lg">
    <h2 class="text-xl font-bold mb-4">Cadastro</h2>

    <!-- Formulário de Cadastro -->
    <form @submit.prevent="submitForm">
      <div class="mb-4">
        <label class="block text-gray-700">E-mail</label>
        <input v-model="form.email" type="email" class="w-full p-2 border rounded" required />
      </div>

      <div class="mb-4">
        <label class="block text-gray-700">Senha</label>
        <input v-model="form.password" type="password" class="w-full p-2 border rounded" required />
      </div>

      <button type="submit" class="bg-blue-500 text-white px-4 py-2 rounded">Cadastrar</button>
    </form>

    <!-- Lista de Usuários -->
    <h2 class="text-xl font-bold mt-6">Usuários Cadastrados</h2>
    <ul class="mt-4">
      <li v-for="user in users" :key="user.id" class="border-b py-2 flex justify-between">
        <span>{{ user.email }}</span>
        <div>
          <button @click="updateUser(user.id, prompt('Novo e-mail:', user.email))" class="text-blue-500">Editar</button>
          <button @click="deleteUser(user.id)" class="text-red-500 ml-2">Excluir</button>
        </div>
      </li>
    </ul>
  </div>
</template>

<script>
import { initializeApp } from "firebase/app";
import { getFirestore, collection, addDoc, getDocs, updateDoc, deleteDoc, doc, setDoc } from "firebase/firestore";
import { getAuth, createUserWithEmailAndPassword } from "firebase/auth";

// Configuração do Firebase
const firebaseConfig = {
  apiKey: "AIzaSyCe0z0Hc54XwXEdRP8AxK7lNoFmu0CRU8o",
  authDomain: "mercado-5ddd2.firebaseapp.com",
  projectId: "mercado-5ddd2",
  storageBucket: "mercado-5ddd2.appspot.com",
  messagingSenderId: "937547819873",
  appId: "1:937547819873:web:84fa3fc8594a325712f320",
  measurementId: "G-QP73JKXE8S"
};

// Inicializa Firebase e Firestore
const app = initializeApp(firebaseConfig);
const db = getFirestore(app);
const auth = getAuth();

export default {
  data() {
    return {
      form: {
        email: '',
        password: ''
      },
      users: [] 
    };
  },
  methods: {
    async submitForm() {
      try {
        // Criar usuário na autenticação do Firebase
        const userCredential = await createUserWithEmailAndPassword(auth, this.form.email, this.form.password);
        const user = userCredential.user;

        // Salvar usuário no Firestore
        await setDoc(doc(db, "usuarios", "users", "userDocs", user.uid), {
          email: this.form.email,
          uid: user.uid,
          createdAt: new Date()
        });

        alert('Usuário cadastrado com sucesso!');
        this.form.email = '';
        this.form.password = '';

        this.fetchUsers(); // Atualiza a lista
      } catch (error) {
        if (error.code === "auth/email-already-in-use") {
          alert("Este e-mail já está cadastrado. Tente outro!");
        } else {
          console.error("Erro ao cadastrar:", error);
          alert(error.message);
        }
      }
    },


    async fetchUsers() {
      const querySnapshot = await getDocs(collection(db, "usuarios", "users", "userDocs"));
      this.users = querySnapshot.docs.map(doc => ({
        id: doc.id,
        ...doc.data()
      }));
    },

    // Atualizar e-mail do usuário no Firestore
    async updateUser(userId, newEmail) {
      if (!newEmail) return; // Se o usuário cancelar, não faz nada

      const userRef = doc(db, "usuarios", "users", "userDocs", userId);
      await updateDoc(userRef, { email: newEmail });

      alert("E-mail atualizado!");
      this.fetchUsers(); // Atualiza a lista
    },

    async deleteUser(userId) {
  try {
    //  Excluir o usuário do Firestore
    const userRef = doc(db, "usuarios", "users", "userDocs", userId);
    await deleteDoc(userRef);

    alert("Usuário deletado com sucesso!");
    this.fetchUsers(); 
  } catch (error) {
    console.error("Erro ao excluir usuário:", error);
    alert("Erro ao excluir. Verifique o console.");
  }
}
  },

  // Busca os usuários assim que a página carregar
  mounted() {
    this.fetchUsers();
  }
};
</script>