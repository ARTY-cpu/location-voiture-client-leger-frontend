<!-- AppOptions.vue -->

<template>
  <div class="container">
    <div class="card mt-5">
      <div class="card-body">
        <h2 class="card-title">Options : Modifier son profil</h2>

        <!-- Affiche le message de succès si la modification a réussi -->
        <div v-if="modificationReussie" class="alert alert-success" role="alert">
          Félicitations ! Vous avez modifié les informations avec succès.
        </div>

        <!-- Affiche les données de l'utilisateur -->
        <div v-if="userData">
          <div class="mb-3">
            <label for="nom" class="form-label">Nom :</label>
            <input type="text" class="form-control" id="nom" v-model="userData.nom" required>
          </div>
          <div class="mb-3">
            <label for="prenom" class="form-label">Prénom :</label>
            <input type="text" class="form-control" id="prenom" v-model="userData.prenom" required>
          </div>
          <div class="mb-3">
            <label for="adresse" class="form-label">Adresse :</label>
            <input type="text" class="form-control" id="adresse" v-model="userData.adresse" required>
          </div>
          <div class="row mb-3">
            <div class="col-md-6">
              <label for="codePostal" class="form-label">Code Postal :</label>
              <input type="text" class="form-control" id="codePostal" v-model="userData.code_postal" required>
            </div>
            <div class="col-md-6">
              <label for="ville" class="form-label">Ville :</label>
              <input type="text" class="form-control" id="ville" v-model="userData.ville" required>
            </div>
          </div>
          <div class="mb-3">
            <label for="telephone" class="form-label">Numéro de téléphone :</label>
            <input type="tel" class="form-control" id="telephone" v-model="userData.telephone" required>
          </div>
          <div class="mb-3">
            <label for="email" class="form-label">Email : <i>(Non modifiable)</i></label>
            <input type="email" class="form-control" id="email" v-model="userData.email" disabled>
          </div>
          <div class="mb-3">
            <label for="mdp" class="form-label">Mot de Passe : <i>(Obligatoire)</i></label>
            <div class="input-group">
              <input :type="showPassword ? 'text' : 'password'" class="form-control" id="mdp" v-model="userData.mdp"
                required />
              <div class="input-group-append">
                <div class="input-group-text">
                  <input type="checkbox" id="showPasswordCheckbox" v-model="showPassword"
                    @change="togglePasswordVisibility" />
                  <label for="showPasswordCheckbox" class="form-check-label">Afficher le mot de passe</label>
                </div>
              </div>
            </div>
          </div>
          <!-- Ajout champs -->

          <!-- Bouton de modification -->
          <button type="submit" class="btn btn-primary" @click="submitForm">Modifier</button>
          <!-- Bouton de désactivation du compte -->
          <button type="button" class="btn btn-danger mx-2" @click="confirmDesactivation">Désactiver le compte</button>

        </div>
      </div>
    </div>
  </div>
</template>

<script>
import axios from 'axios';
/* eslint-disable no-unused-vars */
//necessaire pour les erreurs
export default {
  data() {
    return {
      modificationReussie: false,
      userData: null,
      token: localStorage.getItem('token'), // recup jwt
      showPassword: false,
    };
  },
  methods: {
    loadUserData() {
      // Effectuer la requête pour récupérer les données de l'utilisateur
      fetch('http://localhost:3000/user', {
        method: 'GET',
        headers: {
          'Authorization': 'Bearer ' + this.token,
        },
      })
        .then(response => response.json())
        .then(data => {
          // Mettre à jour les données du composant avec les données de l'utilisateur
          this.userData = data;
        })
        .catch(error => {
          console.error('Erreur lors de la récupération des données de l\'utilisateur :', error);
        });
    },

    submitForm() {
      // Logique pour soumettre les modifications du formulaire
      const { nom, prenom, adresse, code_postal, ville, telephone, mdp } = this.userData;

      fetch('http://localhost:3000/user', {
        method: 'PUT',
        headers: {
          'Content-Type': 'application/json',
          'Authorization': 'Bearer ' + this.token,
        },
        body: JSON.stringify({
          nom,
          prenom,
          adresse,
          codePostal: code_postal,
          ville,
          telephone,
          mdp,
        }),
      })
        .then(response => response.json())
        .then(data => {
          // Mettre à jour le message de succès si la modification réussit
          this.modificationReussie = true;
        })
        .catch(error => {
          console.error('Erreur lors de la modification des données de l\'utilisateur :', error);
        });
    },

    confirmDesactivation() {
      const isConfirmed = window.confirm("Êtes-vous sûr de vouloir désactiver votre compte ? Cette action est irréversible.");
      if (isConfirmed) {
        this.desactiverCompte();
      }
    },
    async logout() {
      // Nettoyer état de connexion et rediriger utilisateur
      localStorage.removeItem('token');
      this.$store.commit('user/setLoggedIn', false);
      this.$router.push('/connexion');
      alert("Votre compte a été désactivé. Vous avez été déconnecté.");
    },

    async desactiverCompte() {
      try {
        const response = await axios.post('http://localhost:3000/desactiver-compte', {
          mdp: this.userData.mdp,
        }, {
          headers: {
            'Authorization': `Bearer ${this.token}`,
            'Content-Type': 'application/json',
          },
        });

        if (response.status === 200) {
          // methode pour se déco
          this.logout();
        } else {
          // erreurs desactivation compte
          console.error('Erreur lors de la désactivation du compte :', response.statusText);
        }
      } catch (error) {
        // Gérer les erreurs d'Axios
        console.error('Erreur lors de la désactivation du compte :', error.message);
      }
    }
  },
  togglePasswordVisibility() {
    // mdp visible ou non
    this.showPassword = !this.showPassword;
  },



  mounted() {
    // charger données lors de l'appel composant
    this.loadUserData();
  },
};
</script>
