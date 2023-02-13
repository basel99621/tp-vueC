<template>
  <main>
    <div>
      <!-- Un formulaire pour saisir les valeurs de la catégorie à ajouter -->
      <form @submit.prevent="ajouteProduit">
        <div>
          <input id="libelle" v-model="data.formulaireProduit.nom" placeholder="Le nom du produit" />
        </div>
        <div>
          <input id="description" v-model="data.formulaireProduit.prixUntaire" placeholder="Le prix unitaire" />
        </div>
        <button type="submit">Ajouter</button>
      </form>
    </div>
    <div>
      <table>
        <caption>Liste des produits - page {{data.page.number + 1}} / {{data.page.totalPages}}</caption>
        <tr>
          <th>Nom</th>
          <th>Prix</th>
          <th>Stock</th>
          <th>Commandé</th>
        </tr>
        <!-- Si le tableau des catégories est vide -->
        <tr v-if="!data.listeProduits">
          <td colspan="4">Veuillez patienter, chargement des catégories...</td>
        </tr>
        <!-- Si le tableau des catégories n'est pas vide -->
        <tr v-for="produit in data.listeProduits" :key="produit.reference">
          <td>{{ produit.nom }}</td>
          <td>{{ produit.prixUnitaire }}</td>
          <td>{{ produit.unitesEnStock }}</td>
          <td>{{ produit.unitesCommandees }}</td>
          <td>
            <button @click="firstPage(data.links.first.href)">
              La première page
            </button>
          </td>
          <td>
            <button @click="previousPage(data.links.prev.href)">
              ←
            </button>
          </td>
          <td>
            <button @click="nextPage(data.links.next.href)">
              →
            </button>
          </td>
          <td>
            <button @click="lastPage(data.links.last.href)">
              la dernière page
            </button>
          </td>
        </tr>
      </table>
    </div>
  </main>
</template>

<script setup>
import { reactive, onMounted } from "vue";
import { BACKEND, doAjaxRequest } from "../api";

// Pour réinitialiser le formuaire
const produitVide = {
  nom: "",
  prixUntaire: ""
};

const currentPage=ref(0);

let data = reactive({
  // Les données saisies dans le formulaire
  formulaireCategorie: { ...produitVide },
  // La liste des produits affichés sous forme de table
  listeProduits: [],
  links: {},
  page: {}
});

function chargeProduits() {
  // Appel à l'API pour avoir la liste des catégories
  // Trié par code, descendant
  // Verbe HTTP GET par défaut
  doAjaxRequest(BACKEND + "/api/produits?page=0&size=5")
      .then((json) => {
        console.log(json._embedded.produits)
        data.listeProduits = json._embedded.produits;
        data.links = json._links;
        data.page = json.page;
      })
      .catch((error) => alert(error.message));
}
onMounted(() => {
  chargeProduits()
})

function ajouteProduit() {
  // Ajouter une catégorie avec les données du formulaire
  const options = {
    method: "POST", // Verbe HTTP POST pour ajouter un enregistrement
    body: JSON.stringify(data.formulaireProduit),
    headers: {
      "Content-Type": "application/json",
    },
  };
  doAjaxRequest(BACKEND + "/api/categories", options)
      .then(() => {
        // Réinitialiser le formulaire
        data.formulaireProduit = { ...produitVide };
        // Recharger la liste des catégories
        chargeProduits();
      })
      .catch((error) => alert(error.message));
}

function nextPage(nextLink) {
  currentPage.value += 1;
  console.log(nextLink)
  doAjaxRequest(nextLink)
      .then((json) => {
        console.log(json._embedded.produits)
        data.listeProduits = json._embedded.produits;
        data.links = json._links;
        data.page = json.page;
      })
      .catch((error) => alert(error.message));
}
function previousPage(previousLink) {
  currentPage > 0 ? currentPage.value -= 1 : currentPage.value = 0;
  doAjaxRequest(previousLink)
      .then((json) => {
        console.log(json._embedded.produits)
        data.listeProduits = json._embedded.produits;
        data.links = json._links;
        data.page = json.page;
      })
      .catch((error) => alert(error.message));
}
function firstPage(firstLink) {
  currentPage.value = 0
  doAjaxRequest(firstLink)
      .then((json) => {
        console.log(json._embedded.produits)
        data.listeProduits = json._embedded.produits;
        data.links = json._links;
        data.page = json.page;
      })
      .catch((error) => alert(error.message));
}

function lastPage(lastLink) {
  doAjaxRequest(lastLink)
      .then((json) => {
        console.log(json._embedded.produits)
        data.listeProduits = json._embedded.produits;
        data.links = json._links;
        data.page = json.page;
      })
      .catch((error) => alert(error.message));
}


</script>


<style scoped>
td,
th {
  border: 1px solid #ddd;
  padding: 8px;
}


th {
  padding-top: 12px;
  padding-bottom: 12px;
  text-align: left;
  background-color: #232623;
  color: rgb(255, 255, 255);
}
</style>