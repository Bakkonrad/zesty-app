<template>
    <div class="container mt-3 mb-5">
        <Alert v-if="recipeDeleted === false" :type="'danger'"
            :message="'Coś poszło nie tak! Przepis nie został usunięty.'"></Alert>
        <!-- <Alert v-if="!recipe" :type="'danger'" :message="'Nie znaleziono przepisu.'"></Alert> -->
        <div v-if="recipe.id != 0" class="card">
            <div class="card-body">
                <div class="row">
                    <div class="col mb-5">
                        <button @click="$router.go(-1)" class="btn btn-outline-zesty">Poprzednia strona</button>
                    </div>
                    <div v-if="recipe.userId !== getUser()" class="col">
                        </div>
                    <div v-else class="col">
                        <div class="d-flex flex-row-reverse">
                            <button class="btn btn-dismiss" type="button" data-bs-toggle="modal"
                                data-bs-target="#confirmDeleteModal">Usuń</button>
                            <button class="btn btn-outline-zesty me-2" type="button" @click="editRecipe">Edytuj</button>
                        </div>
                        <div class="modal fade" id="confirmDeleteModal" tabindex="-1" aria-labelledby="exampleModalLabel"
                            aria-hidden="true">
                            <div class="modal-dialog">
                                <div class="modal-content">
                                    <div class="modal-header">
                                        <h2 class="fs-5" id="exampleModalLabel">Potwierdź działanie</h2>
                                        <button type="button" class="btn-close" data-bs-dismiss="modal"
                                            aria-label="Close"></button>
                                    </div>
                                    <div class="modal-body">
                                        Czy na pewno chcesz usunąć ten przepis?
                                    </div>
                                    <div class="modal-footer">
                                        <!-- <button type="button" class="btn btn-secondary"
                                            data-bs-dismiss="modal">Anuluj</button> -->
                                        <button type="button" class="btn btn-dismiss" data-bs-dismiss="modal"
                                            @click="deleteRecipe">Usuń przepis</button>
                                    </div>
                                </div>
                            </div>
                        </div>
                    </div>
                </div>
                <div class="d-flex justify-content-center">
                    <div class="col-lg-12 col-md-12">
                        <div class="row row-cols-1 row-cols-lg-2 mb-4">
                            <div class="col">
                                <h1>{{ recipe.title }}</h1>
                                <hr class="mb-0">
                                <div class="d-flex justify-content-between p-1 mb-0">
                                    <div class="col text-center">
                                        <p>Data utworzenia:</p>
                                        <p class="binded-data-frame">{{ formattedDate(recipe.createdAt) }}</p>
                                    </div>
                                    <div class="col text-center">
                                        <p>Autor:</p>
                                        <p class="binded-data-frame">{{ recipe.username }}</p>
                                    </div>
                                    <div class="col text-center">
                                        <p>Ocena: </p>
                                        <star-rating :max="max" :rating="recipe.averageRating" :recipe-id="recipeId"
                                            @addRating="handleAddRating"></star-rating>
                                    </div>
                                    <div class="col text-center">
                                        <p>Typ dania:</p>
                                        <p class="binded-data-frame">{{ recipe.mealTypeName }}</p>
                                    </div>
                                </div>
                                <hr class="mb-4 mt-0">
                                <div class="d-flex flex-column p-1">
                                    <h2>Składniki</h2>
                                    <!-- <p>{{ recipe.Ingredients.split(",") }}</p> -->
                                    <ul class="list-group mb-4">
                                        <li class="list-group-item"
                                            v-for="(ingredient, index) in recipe.ingredients.split(',')">
                                            <input class="form-check-input me-1" type="checkbox" value=""
                                                :id="'checkbox' + index" @change="$event.target.blur()">
                                            <label class="form-check-label stretched-link" :for="'checkbox' + index">{{
                                                ingredient }}</label>
                                        </li>
                                    </ul>
                                    <h2>Instrukcje</h2>
                                    <!-- <p>{{ recipe.Instructions }}</p> -->
                                    <ol class="list-group list-group-numbered mb-4">
                                        <li class="list-group-item" v-for="instruction in recipe.instructions.split('. ')">
                                            {{ instruction }}
                                        </li>
                                    </ol>
                                </div>

                            </div>
                            <div class="col text-center">
                                <img :src="recipe.imageUrl" class="img-fluid mb-2" :alt="recipe.title">
                                <h2>Opis przepisu</h2>
                                <p>{{ recipe.description }}</p>
                            </div>
                        </div>
                        <hr>
                        <div class="row row-cols-1 row-cols-lg-2">
                            <div class="col">
                                <h2>Komentarze</h2>
                                <div class="row mb-2" v-for="comment in sortedComments">
                                    <div class="col">
                                        <div class="card">
                                            <!-- <p>{{ recipe.Comments }}</p> -->
                                            <div class="card-body">
                                                <div class="card-title">
                                                    <h5>Dodane przez: {{ comment.username }}</h5>
                                                </div>
                                                <p class="card-text fst-italic p-2">
                                                    {{ comment.content }}
                                                </p>
                                            </div>
                                        </div>
                                    </div>
                                </div>
                            </div>
                            <div v-if="getUser() != null && getUser() != undefined && getUser() != 0" class="col mt-1">
                                <div class="d-flex flex-column p-0">
                                    <h3>Dodaj własny komentarz</h3>
                                    <textarea style="height: 100px" class="form-control mb-2" placeholder="Dodaj komentarz"
                                        aria-label="Dodaj komentarz" aria-describedby="buttonAddComment"
                                        v-model="commentText"></textarea>
                                    <span class="d-flex flex-row-reverse">
                                        <button class="btn btn-outline-zesty" type="button" id="buttonAddComment"
                                            @click="addComment(commentText)">Dodaj</button>
                                    </span>
                                </div>
                            </div>
                        </div>
                    </div>
                </div>
            </div>
        </div>
    </div>
</template>

<script>
import recipesService from '../recipesService.js';
import StarRating from '../components/StarRating.vue';
import Alert from '../components/Alert.vue';

export default {
    name: 'RecipePage',
    components: { StarRating, Alert },
    props: {
        Id: {
            type: Number
        },
        max: {
            type: Number,
            default: 5
        }
    },
    computed: {
        sortedComments() {
        if (this.recipe && this.recipe.Comments) {
            return this.recipe.Comments.sort((a, b) => {
                return new Date(b.createdAt) - new Date(a.createdAt);
            });
        } else {
            return [];
        }
    }},
    data() {
        return {
            recipe: {
                id: 0,
                title: '',
                mealTypeName: '',
                description: '',
                ingredients: '',
                instructions: '',
                imageUrl: '',
                userId: 0,
                username: '',
                createdAt: '',
                averageRating: 0,
                Comments: []
            },
            // averageRating: 0,
            recipeDeleted: null,
            recipeId: null,
            ratingAdded: null,
            commentAdded: null,
            commentText: ''
        }
    },
    mounted() {
        this.loadRecipe();
    },
    async created() {
        this.recipeId = Number(this.$route.params.Id);
    },
    methods: {
        async loadRecipe() {
            try {
                let recipes = await recipesService.LoadAllRecipes();
                let routeId = Number(this.$route.params.Id);
                this.recipe = recipes.find((recipe) => recipe.id === routeId);
                this.recipe.averageRating = await this.loadAverageRating(this.recipe.id);
                this.loadComments();
            } catch (error) {
                console.log(error);
            }
        },
        formattedDate(date) {
            var formatDate = new Date(date);
            return `${formatDate.getFullYear()}-${formatDate.getMonth() + 1}-${formatDate.getDate()}`;
        },
        async loadAverageRating(recipeId) {
            try {
                var aaverageRating = await recipesService.LoadAverageRating(recipeId);
                return aaverageRating;
            } catch (error) {
                console.log(error);
            }
        },
        async handleAddRating(rating) {
            try {
                // console.log('handleAddRating was called with rating: ' + rating);
                this.ratingAdded = await recipesService.addRating(this.recipe.id, rating, 3);
                if (this.ratingAdded === true) {
                    this.recipe.averageRating = await this.loadAverageRating(this.recipe.id);
                    console.log('Ocena została dodana.');
                    this.$router.push( {name: 'RecipePage', params: { Id: this.recipe.id }});
                }
                else {
                    console.log('Nie udało się dodać oceny.');
                }
            } catch (error) {
                console.log(error);
            }
        },
        editRecipe() {
            this.$router.push({ name: 'EditRecipePage', params: { Id: this.recipe.id } });
        },
        async deleteRecipe() {
            this.recipeDeleted = await recipesService.DeleteRecipe(this.recipe.id);
            if (this.recipeDeleted) {
                this.$router.push({ path: '/myRecipes' });
            }
            else {
                console.log('Nie udało się usunąć przepisu.');
            }
        },
        async loadComments() {
            try {
                let comments = await recipesService.loadComments(this.recipe.id);
                this.recipe.Comments = comments;
            } catch (error) {
                console.log(error);
            }
        },
        async addComment(comment) {
            try {
                this.commentAdded = await recipesService.addComment(this.recipe.id, comment);
                if (this.commentAdded === true) {
                    console.log('Komentarz został dodany.');
                    this.commentText = '';
                    this.loadRecipe();
                }
                else {
                    console.log('Nie udało się dodać komentarza.');
                }
            } catch (error) {
                console.log(error);
            }
        },
        getUser() {
            let user = Number(localStorage.getItem('user'));
            console.log(user);
            return user;
        }
    }
}
</script>

<style scoped>
h1 {
    font-size: 3.8rem;
}

h2 {
    font-size: 1.8rem;
}

h3 {
    font-size: 1.5rem;
}

img {
    max-width: 80%;
    height: auto;
    border-radius: 3%;
    box-shadow: 1px 1px 5px 1px rgba(0, 0, 0, 0.2);
}

.binded-data-frame {
    font-size: 1.3rem;
    font-weight: 600;
    color: #7D8A51;
}

#buttonAddComment {
    width: 30%;

}

.form-check-input:checked {
    background-color: #7D8A51 !important;
    border-color: #1a2a157d;
    box-shadow: 0 0 8px #1a2a15 !important;
}

.checked {
    color: #7D8A51;
}

#starIcon {
    color: #7D8A51 !important;
}
</style>