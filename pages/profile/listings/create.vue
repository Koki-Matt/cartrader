<script setup>
definePageMeta({
  layout: "custom",
  middleware: ["auth"],
});

const { makes } = useCars();
const user = useSupabaseUser();
const supabase = useSupabaseClient();

const info = useState("adInfo", () => {
  return {
    make: "",
    model: "",
    year: "",
    miles: "",
    price: "",
    city: "",
    seats: "",
    features: "",
    description: "",
    image: null,
  };
});

const errorMessage = ref("");

const onChangeInput = (data, name) => {
  info.value[name] = data;
};

const inputs = [
  {
    id: 1,
    title: "Model *",
    name: "model",
    placeholder: "Civic",
  },
  {
    id: 2,
    title: "Year *",
    name: "year",
    placeholder: "2019",
  },
  {
    id: 3,
    title: "Price *",
    name: "price",
    placeholder: "10000",
  },
  {
    id: 4,
    title: "Miles *",
    name: "miles",
    placeholder: "10000",
  },
  {
    id: 5,
    title: "City *",
    name: "city",
    placeholder: "Austin",
  },
  {
    id: 6,
    title: "Number of Seats *",
    name: "seats",
    placeholder: "5",
  },
  {
    id: 7,
    title: "Features *",
    name: "features",
    placeholder: "Leather Interior, No Accidents",
  },
];

const isButtonDisabled = computed(() => {
  for (let key in info.value) {
    if (!info.value[key]) return true;
  }
  return false;
});

const handleSubmit = async () => {
  const fileName = Math.floor(Math.random() * 1000000000000000);
  const { data, error } = await supabase.storage
    .from("images")
    .upload("public/" + fileName, info.value.image);
  if (error) {
    return (errorMessage.value = "Cannot upload image");
  }
  const body = {
    ...info.value,
    city: info.value.city.toLowerCase(),
    features: info.value.features.split(", "),
    numberOfSeats: parseInt(info.value.seats),
    miles: parseInt(info.value.miles),
    price: parseInt(info.value.price),
    year: parseInt(info.value.year),
    name: `${info.value.make} ${info.value.model}`,
    listerId: user.value.id,
    image: data.path,
  };

  delete body.seats;

  try {
    const response = await $fetch("/api/car/listings", {
      method: "post",
      body,
    });
    navigateTo("/profile/listings");
  } catch (err) {
    errorMessage.value = err.statusMessage;
    await supabase.storage.from("images").remove(data.path);
  }
};
</script>

<template>
  <div>
    <div class="mt-24">
      <h1 class="text-6xl">Create a new Listing</h1>
    </div>
    <div class="shadow rounded p-3 mt-5 flex flex-wrap justify-between">
      <CarAdSelect
        title="Make *"
        :options="makes"
        name="make"
        @changeInput="onChangeInput"
      />
      <CarAdInput
        v-for="input in inputs"
        :key="input.id"
        :title="input.title"
        :name="input.name"
        :placeholder="input.placeholder"
        @changeInput="onChangeInput"
      />
      <CarAdTextarea
        title="Description *"
        name="description"
        placeholder=""
        @changeInput="onChangeInput"
      />
      <CarAdImage title="Image *" @changeInput="onChangeInput" />
      <div>
        <button
          :disabled="isButtonDisabled"
          @click="handleSubmit"
          class="bg-blue-400 text-white rounded py-2 px-7 mt-3"
        >
          Submit
        </button>
        <p v-if="errorMessage" class="mt-3 text-red-400">{{ errorMessage }}</p>
      </div>
    </div>
  </div>
</template>
