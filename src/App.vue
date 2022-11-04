<template>
  <div class="container container-lg">
    <nav class="bg-danger p-3 text-light text-center">
      <h1>TAF-UNIGRAN</h1>
    </nav>

    <section class="d-flex align-items-center justify-content-around m-3">
      <h1 class="text-danger">Cadastro de Usuário</h1>
      <button
        type="button"
        class="btn btn-danger text-light float-end"
        @click="addUser(true)"
      >
        <i class="fas fa-user m-2"></i>
        <span>Adicionar</span>
      </button>
    </section>

    <!-- ADICIONAR E EDITAR -->
    <div
      id="overlay"
      v-if="isModalCreateOrEdit"
      class="mx-auto p-4 shadow-lg border-danger rounded"
      style="width: 600px; height: 84%"
    >
      <div class="modal-dialog">
        <div class="modal-content">
          <div class="modal-header">
            <div class="modal-title">
              <h4 class="text-primary text-danger">{{ user & user.id ? "Editar" : "Adicionar"}} usuário </h4>
            </div>
            <button
              class="button btn-close p-2"
              data-bs-dismiss="modal"
              aria-label="Close"
              @click="openModal(false)"
            ></button> 
          </div>
          <div class="modal-body">
            <form @submit.prevent="createUser">
              <div class="formgroup gap-3 p-2">
                <input
                  type="text"
                  class="form-control"
                  v-model="user.nome"
                  placeholder="Nome"
                  aria-label="nome"
                />
              </div>
              <div class="form group gap-3 p-2">
                <input
                  type="text"
                  class="form-control"
                  v-mask="'###.###.###-##'"
                  v-model="user.cpf"
                  maxlength="14"
                  placeholder="CPF"
                />
              </div>
              <div class="form-group gap-3 p-2">
                <input
                  type="text"
                  class="form-control"
                  v-model="user.telefone"
                  maxlength="20"
                  v-mask="['(##) #####-####', '(##) ####-####']"
                  placeholder="Telefone"
                  aria-label="telefone"
                />
              </div>
              <div class="form-group gap-3 p-2">
                <input
                  type="email"
                  class="form-control"
                  v-model="user.email"
                  placeholder="E-mail"
                  aria-label="email"
                />
              </div>
              <div class="form-group gap-3 p-2">
                <input
                  type="date"
                  class="form-control"
                  maxlength="11"
                  v-model="user.data_nascimento"
                  placeholder="Data de Nascimento"
                  aria-label="data_nascimento"
                />
              </div>
              <div class="form-group gap-3 p-2">
                <input
                  type="text"
                  class="form-control"
                  v-model="user.genero"
                  placeholder="Gênero M ou F"
                  aria-label="gênero"
                />
              </div>
              <div class="form-group gap-3 p-2">
                <input
                  type="text"
                  class="form-control"
                  placeholder="Seu Diferencial"
                  v-model="user.seu_diferencial"
                  aria-label="seu_diferencial"
                />
              </div>
              <div class="d-grid gap-2 m2">
                <button type="submit" class="btn btn-success">SALVAR</button>
              </div>
            </form>
          </div>
        </div>
      </div>
    </div>

    <!-- DELETAR -->
    <div id="overlay" v-if="isModalDelete" class="mx-auto p-4 shadow-lg border-danger rounded" 
    style="width: 600px; height: 42%;">
      <div class="modal-dialog">
        <div class="modal-content">
          <div class="modal-header">
            <div class="modal-title">
              <h4 class="text-danger">
                <i class="fas fa info circle text-danger"></i>
                Atenção
              </h4>
            </div>
            <button type="button" class="btn-close" data-bs-dismiss="modal" aria-label="Colse" @click="isModalDelete = false"></button>
          </div>

          <div class="modal-body">
            <h3 class="text-secondary">Deseja remover este usuário
            <span class="rounded">
               {{ user.nome }} 
            </span> 
            ?
          </h3>
          <br/>
          <p class="text-dark">Esta ação não poderá ser refeita.</p>
          <br/>
          </div>
          <div class="modal-footer">
            <button class="btn btn-light" data-bs-dismiss="modal">Cancelar</button>
            <button class="btn btn-danger" data-bs-dismiss="modal">Deletar</button>
          </div>
        </div>
      </div>
    </div>

    <table class="table table-striped table-hover table-bordered">
      <thead>
        <tr class="align-center bg-danger text-light text-center">
          <th>ID</th>
          <th>NOME</th>
          <th>CPF</th>
          <th>TELEFONE</th>
          <th>EMAIL</th>
          <th>DATA DE NASCIMENTO</th>
          <th>GENERO</th>
          <th>SEU DIFERENCIAL</th>
          <th>Editar</th>
          <th>Deletar</th>
        </tr>
      </thead>

      <tbody>
        <tr class="text-center" v-for="user in users">
          <td>{{ user.id }}</td>
          <td>{{ user.nome }}</td>
          <td>{{ user.cpf }}</td>
          <td>{{ user.telefone }}</td>
          <td>{{ user.email }}</td>
          <td>{{ formatDate(user.data_nascimento) }}</td>
          <td>{{ user.genero }}</td>
          <td>{{ user.seu_diferencial }}</td>
          <td>
            <a href="#" class="btn btn-danger text-light" @click="editUser(user)">
              <i class="fas fa-pencil-alt"></i>
            </a>
          </td>
          <td>
            <a href="#" class="btn btn-info text-light" @click="deleteUser(user)">
              <i class="fas fa-trash-alt"></i>
            </a>
          </td>
        </tr>
      </tbody>
    </table>
  </div>
</template>

<script>
import { api } from "./service/api";
import { mask } from "vue-the-mask";

export default {
  name: "Table",
  directives: { mask },
  data() {
    return {
      isModalCreateOrEdit: false,
      isModalDelete: false,
      user: {
        nome: "",
        cpf: "",
        telefone: "",
        email: "",
        data_nascimento: "",
        genero: "",
        seu_diferencial: "",
      },
      users: [],
    };
  },
  mounted() {
    this.getUsers();
  },
  methods: {
    getUsers() {
      api.get("tecnico").then((res) => {
        this.users = res.data;
      });
    },
    createUser() {
      api.post ("tecnico", {
          nome: this.user.nome,
          cpf: this.user.cpf,
          telefone: this.user.telefone,
          email: this.user.email,
          data_nascimento: this.user.data_nascimento,
          genero: this.user.genero,
          seu_diferencial: this.user.seu_diferencial,
        })
          this.isModalCreateOrEdit = false;
          this.editUser = true;
    },
    openModal (isModal){
      this.isModalCreateOrEdit = isModal;
    },
    addUser (isModal){
      this.user = {};
      this.openModal(isModal);
    },
    closeModal (isModal){
      this.openModal = isModal;
    },
    editUser (user){ 
      api.put (`tecnico/${user.id}`, user)
      if (user){
        this.user = user;
        this.openModal(true);
      }
    },
    deleteUser(user){
      api.delete (`tecnico/${user.id}`, user)
      if(user) {
        this.user = user;
        this.isModalDelete = true;
      }
    },
    formatDate(date) {
      const options = { year: "numeric", month: "long", day: "numeric" };
      return new Date(date).toLocaleDateString("pt-br", options);
    },
  },
};
</script>
