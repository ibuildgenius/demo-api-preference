<script setup lang="ts">
import axios from "axios";

const showFormFields = ref(false);
const firstName = ref<string>("");
const lastName = ref<string>("");
const email = ref<string>("");
const phoneNumber = ref<string>("");
const gender = ref<string>("");
const maritalStatus = ref<string>("");
const dateOfBirth = ref<string>("");
const planDuration = ref<string>("");
const idCardUrl = ref<string>("");
const idCardImage = ref<File>();
const isFileSizeError = ref(false);
const isLoading = ref(false);
const isSuccess = ref(false);
const HmoId = ref<string>("");

const handleProceedBtn = () => {
  showFormFields.value = !showFormFields.value;
};

const handleBuyBtn = async () => {
  const productId = "0521ffe3-e7c1-4bfa-b90e-d69ab311ec98";
  const baseUrl = "https://api.mycover.ai/v1";
  const token = ""; // YOUR_SECRET_KEY
  const authToken = `Bearer ${token}`;

  isLoading.value = true;

  // upload file
  if (idCardImage.value) {
    const formData = new FormData();
    formData.append("file", idCardImage.value);

    const response = await axios.post(`${baseUrl}/upload-file`, formData, {
      headers: {
        "Content-Type": "multipart/form-data",
        Authorization: authToken,
      },
    });

    if (response) {
      idCardUrl.value = response.data?.data?.file_url;

      // construct the payload
      const payload = {
        first_name: firstName.value,
        last_name: lastName.value,
        email: email.value,
        phone_number: phoneNumber.value,
        gender: gender.value,
        marital_status: maritalStatus.value,
        image_url: idCardUrl.value,
        date_of_birth: dateOfBirth.value,
        payment_plan: planDuration.value,
        product_id: productId,
      };

      // then submit the payload
      const result = await axios.post(
        `${baseUrl}/products/bastion/buy-health`,
        payload,
        {
          headers: {
            Accept: "application/json",
            "Content-Type": "application/json;charset=UTF-8",
            Authorization: authToken,
          },
        }
      );

      isLoading.value = false;

      if (result) {
        HmoId.value = result.data?.data?.policy?.meta?.hmo_policy_id;
        showFormFields.value = false;
        isSuccess.value = true;
      }
    }
  }
};

const convertBytesToMb = (bytes: number) => {
  return Number((bytes / (1024 * 1024)).toFixed(2));
};

const handleFileUpload = (e: Event) => {
  const target = e.target as HTMLInputElement;
  const file = target.files ? target.files[0] : null;

  if (file) {
    if (convertBytesToMb(file.size) >= 3) {
      isFileSizeError.value = true;
    } else {
      isFileSizeError.value = false;
      idCardImage.value = file;
    }
  }
};
</script>

<template>
  <section class="relative h-screen w-screen">
    <div
      class="mx-auto max-w-2xl px-4 py-16 sm:px-6 sm:py-24 lg:max-w-7xl lg:px-8"
    >
      <h2 class="text-2xl font-bold tracking-tight text-gray-900">
        MyApp - Offers a wide range of health insurance products
      </h2>

      <div
        class="mt-6 grid grid-cols-1 gap-x-6 gap-y-10 sm:grid-cols-2 xl:gap-x-8"
      >
        <p v-if="isSuccess">Congrats! 🎉 This is your HMO ID: {{ HmoId }}</p>
        <div v-else class="group relative">
          <div
            class="aspect-h-1 aspect-w-1 w-full overflow-hidden rounded-md bg-gray-200 lg:aspect-none group-hover:opacity-75 lg:h-80"
          >
            <img
              src="https://i0.wp.com/oyshia.oy.gov.ng/wp-content/uploads/2023/09/health-insurance-terms.jpg"
              alt="Front of men&#039;s Basic Tee in black."
              class="h-full w-full object-cover object-center lg:h-full lg:w-full"
            />
          </div>
          <div class="mt-4 flex justify-between">
            <div @click="handleProceedBtn" class="[&>div]:hover:translate-y-1">
              <button
                id="buy-insurance"
                class="mt-0.5 sm:mt-0 text-prime font-bold"
              >
                Proceed
              </button>
              <div
                class="h-[2px] w-full bg-prime transition duration-300 ease-in-out"
              ></div>
            </div>

            <p class="text-sm font-medium text-gray-900">Bastion</p>
          </div>
        </div>

        <img v-if="isSuccess" src="./assets/img/success.gif" alt="success" />

        <form v-if="showFormFields">
          <div class="mb-4 w-full">
            <label for="first-name">First Name</label>
            <input
              id="first-name"
              class="w-full border px-4 py-2 rounded focus:border focus:border-blue-500 focus:shadow-outline outline-none"
              type="text"
              placeholder="Kenneth"
              required
              v-model="firstName"
            />
          </div>

          <div class="mb-4 w-full">
            <label for="last-name">Last Name</label>
            <input
              id="last-name"
              class="w-full border px-4 py-2 rounded focus:border focus:border-blue-500 focus:shadow-outline outline-none"
              type="text"
              placeholder="Jimmy"
              required
              v-model="lastName"
            />
          </div>

          <div class="mb-4 w-full">
            <label for="email">Email</label>
            <input
              id="email"
              class="w-full border px-4 py-2 rounded focus:border focus:border-blue-500 focus:shadow-outline outline-none"
              type="email"
              placeholder="jimmy@mycovergenius.com"
              required
              v-model="email"
            />
          </div>

          <div class="mb-4 w-full">
            <label for="phone">Phone Number</label>
            <input
              id="phone"
              class="w-full border px-4 py-2 rounded focus:border focus:border-blue-500 focus:shadow-outline outline-none"
              type="tel"
              placeholder="09135628827"
              required
              v-model="phoneNumber"
            />
          </div>

          <div class="mb-4 w-full">
            <label for="payment-plan">Plan Duration</label>
            <input
              id="payment-plan"
              class="w-full border px-4 py-2 rounded focus:border focus:border-blue-500 focus:shadow-outline outline-none"
              type="number"
              required
              v-model="planDuration"
            />
          </div>

          <div class="mb-4 w-full">
            <label class="mr-5" for="gender">Gender:</label>
            <select
              v-model="gender"
              class="w-20 cursor-pointer"
              name="gender"
              id="gender"
            >
              <option value="Male">Male</option>
              <option value="Female">Female</option>
            </select>
          </div>

          <div class="mb-4 w-full">
            <label class="mr-5" for="marital-status">Marital Status:</label>
            <select
              class="w-20 cursor-pointer"
              name="marital-status"
              id="marital-status"
              v-model="maritalStatus"
            >
              <option value="Married">Married</option>
              <option value="Single">Single</option>
            </select>
          </div>

          <div class="mb-4 w-full">
            <label class="block" for="birth-date">Date of Birth:</label>
            <input
              v-model="dateOfBirth"
              type="date"
              id="birth-date"
              name="birth-date"
            />
          </div>

          <div class="mb-4 w-full">
            <label class="block" for="id-card">Upload your ID:</label>
            <input
              type="file"
              id="id-card"
              name="id-card"
              accept="image/png, image/jpeg"
              @change="handleFileUpload"
            />

            <p v-if="isFileSizeError" class="text-danger text-sm">
              Image file size must be less than 3MB
            </p>
          </div>

          <div class="text-right">
            <div
              @click.prevent="handleBuyBtn"
              class="[&>div]:hover:translate-y-1 inline-block"
            >
              <div v-if="isLoading" class="loader"></div>
              <button
                v-else
                id="buy-insurance"
                class="mt-0.5 sm:mt-0 text-prime font-bold"
                type="submit"
              >
                Buy
              </button>
              <div
                class="h-[2px] w-full bg-prime transition duration-300 ease-in-out"
              ></div>
            </div>
          </div>
        </form>
      </div>
    </div>
  </section>
</template>

<style lang="css">
.loader {
  border: 4px solid #f3f3f3; /* Light grey */
  border-top: 4px solid #3aaa90; /* Blue */
  border-radius: 50%;
  width: 30px;
  height: 30px;
  animation: spin 2s linear infinite;
}

@keyframes spin {
  0% {
    transform: rotate(0deg);
  }
  100% {
    transform: rotate(360deg);
  }
}
</style>
