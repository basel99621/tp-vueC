<template>
    <main>
        <div>
            <!-- Un formulaire pour saisir les valeurs de la catégorie à ajouter -->
            <form @submit.prevent="ajouteCategorie">
                <div>
                    <input id="libelle" v-model="data.formulaireCategorie.libelle" placeholder="Libelle" />
                </div>
                <div>
                    <input id="description" v-model="data.formulaireCategorie.description" placeholder="Description" />
                </div>
                <button type="submit">Ajouter</button>
            </form>
        </div>
        <div>
            <table>
                <caption>Liste des catégories</caption>
                <tr>
                    <th>Code</th>
                    <th>Libelle</th>
                    <th>Description</th>
                    <th>Action</th>
                </tr>
                <!-- Si le tableau des catégories est vide -->
                <tr v-if="!data.listeCategories">
                    <td colspan="4">Veuillez patienter, chargement des catégories...</td>
                </tr>
                <!-- Si le tableau des catégories n'est pas vide -->
                <tr v-for="categorie in data.listeCategories" :key="categorie.code">
                    <td>{{ categorie.code }}</td>
                    <td>{{ categorie.libelle }}</td>
                    <td>{{ categorie.description }}</td>
                    <td>
                        <button @click="deleteEntity(categorie._links.self.href)">
                            Supprimer
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
const categorieVide = {
    libelle: "",
    description: ""
};

let data = reactive({
    // Les données saisies dans le formulaire
    formulaireCategorie: { ...categorieVide },
    // La liste des catégories affichée sous forme de table
    listeCategories: []
});

function chargeCategories() {
    // Appel à l'API pour avoir la liste des catégories
    // Trié par code, descendant
    // Verbe HTTP GET par défaut
    doAjaxRequest(BACKEND + "/api/categories?sort=code,desc")
        .then((json) => {
            data.listeCategories = json._embedded.categories;
        })
        .catch((error) => alert(error.message));
}

function ajouteCategorie() {
    // Ajouter une catégorie avec les données du formulaire
    const options = {
        method: "POST", // Verbe HTTP POST pour ajouter un enregistrement
        body: JSON.stringify(data.formulaireCategorie),
        headers: {
            "Content-Type": "application/json",
        },
    };
    doAjaxRequest(BACKEND + "/api/categories", options)
        .then(() => {
            // Réinitialiser le formulaire
            data.formulaireCategorie = { ...categorieVide };
            // Recharger la liste des catégories
            chargeCategories();
        })
        .catch((error) => alert(error.message));
}
/**
 * Supprime une entité
 * @param entityRef l'URI de l'entité à supprimer
 */
function deleteEntity(entityRef) {
    doAjaxRequest(entityRef, { method: "DELETE" })
        .then(chargeCategories)
        .catch((error) => alert(error.message));
}

// A l'affichage du composant, on affiche la liste
onMounted(chargeCategories);

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