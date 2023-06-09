<template>
  <div class="d-flex flex-column align-items-center my-4">
    <form id="new-quiz-form" @submit.prevent="launchNewQuiz">
      <h1 class="text-center mb-4">Commencer un nouveau quiz</h1>
      <div class="form-outline mb-4">
        <label class="form-label" for="nom-joueur">Saisissez votre nom : </label>
        <input class="form-control" type="text" id="nom-joueur" name="nom-joueur" placeholder="Votre nom"
          aria-describedby="Le nom qui sera affiché sur la table des scores" v-model="username" required>
      </div>
      <div class="text-center">
        <button type="submit" class="btn btn-primary btn-block" :disabled="error">Commencer !</button>
      </div>
    </form>
    <div v-if="error" class="alert alert-danger my-5" role="alert">
      Aucune question n'est disponible ! Impossible de démarrer un nouveau Quiz.
    </div>
  </div>
</template>

<script>
/**
 * Component: NewQuizPage
 * Description: Represents the page for starting a new quiz.
 *
 * Data:
 *   - username (String): The username of the player.
 *
 * Methods:
 *   - created(): Lifecycle hook called when the component is created. Retrieves the player name from storage and check if the database contains questions.
 *   - launchNewQuiz(): Launches a new quiz by clearing storage, saving the player name, and navigating to the questions page.
 */
import participationStorageService from "@/services/ParticipationStorageService.js";
import quizApiService from "@/services/QuizApiService.js";

export default {
  name: "NewQuizPage",
  data() {
    return {
      username: '',
      error: false
    };
  },
  async created() {
    this.username = participationStorageService.getPlayerName() ?? '';

    quizApiService.getQuizInfo()
            .then((response) => {
                if (response.status !== 200) {
                    const ERROR = `CODE : ${response.status} getQuizInfo`
                    return Promise.reject(ERROR);
                }

                this.error = response.data.size === 0;
            }
            ).catch((error) => {
                console.log(error);
            }
            );
  },
  methods: {
    /**
     * Launches a new quiz by clearing storage, saving the player name, and navigating to the questions page.
     */
    launchNewQuiz() {
      if (!this.username) {
        return;
      }

      participationStorageService.clear();
      participationStorageService.savePlayerName(this.username);
      this.$router.push('/questions');
    }
  }
};
</script>
