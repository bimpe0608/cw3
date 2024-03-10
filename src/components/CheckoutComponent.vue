<template>
    <section v-show="cartPage" class="mt-4 pb-4">
        <div class="d-flex justify-content-center align-content-center">
            <h1>
                This is the cart Page
            </h1>
        </div>

        <div v-for="(lesson, index) in cart" :key="index" class="mx-4">
            <div class="card m-1 p-1">
                <div class="card-body" style="display: flex;">
                    <div>
                        <h5 class="card-title text-left">
                            {{ lesson.subject }}
                        </h5>
                        <h6 class="card-subtitle text-left">
                            {{ lesson.location }}
                        </h6>
                        <p class="card-text">
                            &#163;{{ lesson.price }}
                        </p>
                        <p class="card-text">
                            Availability: {{ lesson.availability }}
                        </p>
                        <button class="btn btn-outline-danger" @click="$emit('remove', lesson)">
                            Remove from cart
                        </button>
                    </div>
                    <div style="margin-left: 2rem; font-size: 3rem; color:blue;">
                        <img :src="lesson.imagePath"
                            height="100" width="90">
                    </div>
                </div>
            </div>
        </div>

        <form class="m-4" @submit.prevent="createOrder">
            <div class="form-group">
                <h2>Create an order</h2>
            </div>
            <div class="form-group m-4">
                <label for="name">Name</label>
                <input type="text" v-model="checkout.name" class="form-control" id="name" @keypress="isLetters($event)"
                    placeholder="Enter name">
            </div>
            <div class="form-group m-4">
                <label for="phoneNumber">Phone Number</label>
                <input type="text" v-model="checkout.phoneNumber" class="form-control" id="phoneNumber"
                    @keypress="isNumbers($event)" placeholder="Phone Number">
            </div>
            <button type="submit" :disabled="checkout.name == '' || checkout.phoneNumber == ''">
                Create Order
            </button>
        </form>
    </section>
</template>



<script>
export default {
    props: ['checkout', 'cart', 'cartPage']
}
</script>