<template>
    <div id="app" class="container">
        <h1 class="d-flex justify-content-center pt-4 pb-2">
            Book Available Lessons
        </h1>


        <!-- Cart Button -->
        <div class="container-fluid m-4">
            <button class="btn btn-outline-success" v-show="cart.length" @click="changeToCartPage">
                <div class="text-center m-2" style="width: 10rem">
                    <p style="font-size: larger;">Cart <i class="fa fa-shopping-cart" aria-hidden="true"></i></p>
                    <span>{{ (cart?.length) }} items</span>
                </div>
            </button>
        </div>
        <!-- Cart Button ends-->




        <!-- Search -->
        <div class="d-flex justify-content-center">
            <form>
                <input type="text" placeholder="search" @input="searchLessons" v-model="searchInput">
            </form>
        </div>
        <!-- Search ends-->

        <!-- Sorting starts-->
        <div class="p-2">
            <div class="d-flex justify-content-center m-2">
                <button class="btn btn-outline-success m-1" @click="sortSubjects">
                    Subject
                </button>
                <button class="btn btn-outline-success m-1" @click="sortLocations">
                    Location
                </button>
                <button class="btn btn-outline-success m-1" @click="sortPrices">
                    Price
                </button>
                <button class="btn btn-outline-success m-1" @click="sortSpaces">
                    Availability
                </button>
            </div>
            <div class="d-flex justify-content-center m-2">
                <button class="btn btn-outline-success m-1" @click="changeToAscending" :class="{ 'btn-success': activeBtn === 'asc' }">
                    <p>Ascending</p>
                </button>
                <button class="btn btn-outline-success m-1" @click="changeToDescending" :class="{ 'btn-success': activeBtn === 'dsc' }">
                    <p>Descending</p>
                </button>
                <button class="btn btn-outline-success m-1" @click="toggleTestConsole">
                    <p>Test Console</p>
                </button>
            </div>
             <div v-if="testConsole" class="d-flex justify-content-center m-2">
                <button class="btn btn-outline-success m-1" @click="reload">Reload Page</button>
                <button class="btn btn-outline-success m-1" @click="delateAllCache">Delete all cache</button> 
                <button class="btn btn-outline-success m-1" @click="unregisterSW">Unregister All Service Workers</button>
            </div>
        </div>
        <!-- Sorting ends-->

       
        <LessonsComponent v-if="!cartPage" :lessons="lessons" @add="addToCart" />
        <CheckoutComponent v-else :checkout="checkout" :cart="cart" :cartPage="cartPage" @remove="removeFromCart" />
    </div>
</template>

<script>
import LessonsComponent from "./components/LessonsComponent.vue"
import CheckoutComponent from "./components/CheckoutComponent.vue"

export default {
    name: 'App',
    components: {
        LessonsComponent,
        CheckoutComponent
    },
    data() {
        return {
            cartPage: false,
            ascending: true,
            descending: false,
            activeBtn: "",
            cart: [],
            checkout: {
                name: "",
                phoneNumber: "",
                lessonId: null,
                numOfSpaces: 0
            },
            lessons: [],
            searchInput: "",
            testConsole: false
        }
    },
    methods: {
        async fetchLessons() {
            try {
                const response = await fetch('http://vuejsbackend.eu-north-1.elasticbeanstalk.com/lessons');
                if (!response.ok) {
                    throw new Error('Failed to fetch lessons');
                }
                this.lessons = await response.json();
            } catch (error) {
                console.error(error);
            }
        },
        async updateLessonSpaces(lessonId, newSpaces) {
            try {
                const response = await fetch(`http://vuejsbackend.eu-north-1.elasticbeanstalk.com/lessons/${lessonId}`, {
                    method: 'PUT',
                    body: JSON.stringify([{ _id: lessonId, availability: newSpaces }]),
                    headers: {
                        'Content-Type': 'application/json',
                    },
                });

                return await response.json();
            } catch (error) {
                console.error(error);
                throw new Error('Failed to update lesson availability');
            }
        },

        async addToCart(product) {
            if (product.availability > 0) {
              // Add the lesson to the cart
              this.cart.push(product);

              // Reduce the remaining space by one
              product.availability -= 1;
            }
        },
        async removeFromCart(product) {
            // Increase the availability of the removed product
            product.availability += 1;

            // Remove the product from the cart
            this.cart.splice(this.cart.indexOf(product), 1);
        },
        async createOrder(order) {
            try {
              console.log(order);

              const url = `${this.url}/orders`;

              const response = await fetch(url, {
                method: "POST",
                headers: { "Content-Type": "application/json" },
                body: JSON.stringify(order),
              });
              return await response.json()
            } catch (error) {
              console.error(`Error creating order.. ${error}`)
            }
        },
        async searchLessons() {
            try {
                // Sending request for fetching lessons based on the searchInput value
                const response = await fetch(`http://vuejsbackend.eu-north-1.elasticbeanstalk.com/lessons?search=${this.searchInput}`);
                this.lessons = await response.json();
            } catch (error) {
                console.error(error);
            }
        },
        clearCart() {
            this.cart = [];
            this.cartPage = false;
        },
        changeToCartPage() {
            this.cartPage = !this.cartPage;
        },
        changeToAscending() {
            this.descending = false;
            this.ascending = true;
            this.activeBtn = 'asc'
        },
        changeToDescending() {
            this.ascending = false;
            this.descending = true;
            this.activeBtn = 'dsc'
        },
        sortPrices() {
            if (this.ascending) {
                let sortedSubs = this.lessons
                sortedSubs = sortedSubs.sort((a, b) => {
                    return a.price - b.price;
                })
            }
            if (this.descending) {
                let sortedSubs = this.lessons
                sortedSubs = sortedSubs.sort((a, b) => {
                    return b.price - a.price
                })
            }
        },
        sortLocations() {
            if (this.ascending) {
                let sortedSubs = this.lessons
                sortedSubs = sortedSubs.sort((a, b) => {
                    let subsA = a.location.toLowerCase()
                    let subsB = b.location.toLowerCase()
                    if (subsA < subsB) {
                        return -1
                    }
                    if (subsA > subsB) {
                        return 1
                    }
                    return 0
                })
            }
            if (this.descending) {
                let sortedSubs = this.lessons
                sortedSubs = sortedSubs.sort((a, b) => {
                    let subsA = a.location.toLowerCase()
                    let subsB = b.location.toLowerCase()
                    if (subsA < subsB) {
                        return 1
                    }
                    if (subsA > subsB) {
                        return -1
                    }
                    return 0
                })
            }
        },
        sortSpaces() {
            if (this.ascending) {
                let sortedSubs = this.lessons
                sortedSubs = sortedSubs.sort((a, b) => {
                    return a.availability - b.availability;
                })
            }
            if (this.descending) {
                let sortedSubs = this.lessons
                sortedSubs = sortedSubs.sort((a, b) => {
                    return b.availability - a.availability;
                })
            }
        },
        sortSubjects() {
            if (this.ascending) {
                let sortedSubs = this.lessons
                sortedSubs = sortedSubs.sort((a, b) => {
                    let subsA = a.subject.toLowerCase()
                    let subsB = b.subject.toLowerCase()
                    if (subsA < subsB) {
                        return -1
                    }
                    if (subsA > subsB) {
                        return 1
                    }
                    return 0
                })
            }
            if (this.descending) {
                let sortedSubs = this.lessons
                sortedSubs = sortedSubs.sort((a, b) => {
                    let subsA = a.subject.toLowerCase()
                    let subsB = b.subject.toLowerCase()
                    if (subsA < subsB) {
                        return 1
                    }
                    if (subsA > subsB) {
                        return -1
                    }
                    return 0
                })
            }
        },
        isLetters($event) {
            var charCode = $event.keyCode;
            if ((charCode > 64 && charCode < 91) || (charCode > 96 && charCode < 123) || charCode == 8) {
                return true;
            } else {
                $event.preventDefault();
            }
        },
        isNumbers($event) {
            var charCode = $event.keyCode;
            if ((charCode > 31 && (charCode < 48 || charCode > 57))) {
                $event.preventDefault();
            } else {
                return true;
            }
        },
        reload() {
            window.location.reload(true);
        },
        delateAllCache() {
            caches.keys().then(function (names) {
                for (let name of names)
                    caches.delete(name);
            });
            console.log("All Caches Deleted");
        },
        unregisterSW() {
            navigator.serviceWorker.getRegistrations().then(function (registrations) {
                    for (let registration of registrations) {
                        registration.unregister()
                    }
                });
                console.log("ServiceWorkers Unregistered");
        },
        toggleTestConsole() {
            this.testConsole = !this.testConsole
        }
    },
    mounted() {
        // Fetch lessons when the app is mounted
        this.fetchLessons();
    },
    // Watch for changes in the 'searchInput' data property
    watch: {
        // When 'searchInput' changes, call the searchLessons method
        searchInput: function () {
            this.searchLessons();
        },
    },
   created: function () {
        if ("serviceWorker" in navigator) {
            navigator.serviceWorker.register("service-worker.js");
        }
    }
}
</script>

<style>
#app {
    font-family: Avenir, Helvetica, Arial, sans-serif;
    -webkit-font-smoothing: antialiased;
    -moz-osx-font-smoothing: grayscale;
    text-align: center;
    color: #2c3e50;
    margin-top: 60px;
}
</style>
