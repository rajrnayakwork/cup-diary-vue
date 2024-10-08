<template>
    <div>
        <Card>
            <template #headerLeft>
                <Label class="m-0 p-0 text-3xl">Users</Label>
            </template>
            <template #headerRight>
                <Button @click="openFormModal" variant="flat"
                    class="border-1 border-blue-600 text-blue-600 hover:bg-blue-600 hover:text-white">
                    Add User
                </Button>
            </template>
            <DataTable :data="users" :columns="dataTable.columns" :search-fields="searchFields" selectColumns
                columnSorting @load-Data="loadData" :args="dataTable.args" :paginate="dataTable.paginate">
                <template #action="actionProps">
                    <div class="d-flex gap-1">
                        <Button @click="() => openFormModal(actionProps.row)" variant="flat"
                            class="border-1 border-green-700 text-green-700 hover:bg-green-700 hover:text-white p-3">
                            <Edit class="h-4 w-4" />
                        </Button>
                        <Button variant="flat"
                            class="border-1 border-red-600 text-red-600 hover:bg-red-600 hover:text-white p-3"
                            @click="() => destroy(actionProps.row.id)">
                            <Trash2 class="h-4 w-4" />
                        </Button>
                    </div>
                </template>
            </DataTable>
        </Card>
        <UserForm ref="formRef" :user="user" @handle-submit="handleSubmit" @close-modal="closeModal" />
    </div>
</template>

<script setup>
import { onMounted, reactive, ref } from "vue";
import { usePage } from "@inertiajs/vue3";
import axios from "axios";
import Card from "../components/Card.vue";
import DataTable from "../components/DataTable.vue";
import UserForm from "./UserForm.vue";
import { Label } from "@/components/ui/label";
import { Button } from "@/components/ui/button";
import { Edit, Trash2 } from "lucide-vue-next";

const { baseUrl } = usePage().props;

const users = ref([]);
const user = ref({
    id: null,
    username: "",
    first_name: "",
    last_name: "",
    email: "",
    mobile_number: "1234567890",
});
const formRef = ref(null);

// let pagination = reactive({
//     current_page: 1,
//     per_page: 5,
//     total_page: 0,
//     from: 0,
//     to: 0,
//     total: 0,
// });

let dataTable = reactive({
    columns: [
        { key: "id", name: "Id" ,isSearch : false},
        { key: "username", name: "Username" ,isSearch : false},
        { key: "first_name", name: "First Name" ,isSearch : false},
        { key: "last_name", name: "Last Name" ,isSearch : false},
        { key: "email", name: "Email" ,isSearch : false},
        {
            key: "action",
            name: "Action",
            renderBody: true,
            isAlwaysShow: true,
            notSorted: true,
        },
    ],
    paginate: {
        perPage: 5,
        total: 0,
    },
    // args: {}
});

// const columns = [
//     { key: "id", name: "Id" },
//     { key: "username", name: "Username" },
//     { key: "first_name", name: "First Name" },
//     { key: "last_name", name: "Last Name" },
//     { key: "email", name: "Email" },
//     {
//         key: "action",
//         name: "Action",
//         renderBody: true,
//         isAlwaysShow: true,
//         notSorted: true,
//     },
// ];

let searchFields = reactive([
    { key: "username", name: "Username", isAllowed: true },
    { key: "first_name", name: "First Name", isAllowed: true },
    { key: "last_name", name: "Last Name", isAllowed: true },
    { key: "email", name: "Email", isAllowed: true },
]);

const loadData = async (
    page = 1,
    perPage = dataTable.paginate.perPage,
    searchArray = {},
    sortedColumns = {}
) => {
    await axios
        .post(`${baseUrl}/user/load-data`, {
            per_page_number: perPage,
            page_number: page,
            search_fields: searchArray,
            sorting_columns: sortedColumns,
        })
        .then((response) => {
            users.value = response.data.data;
            // pagination = {
            //     current_page: page,
            //     per_page: perPage,
            //     total_page: response.data.last_page,
            //     from: response.data.from,
            //     to: response.data.to,
            //     total: response.data.total,
            // };
            dataTable.paginate.total =  response.data.total
        })
        .catch((error) => {
            console.log(error);
        });
};

const openFormModal = (value) => {
    if (value) {
        user.value = value;
    }
    formRef.value.openForm();
};

const handleSubmit = () => {
    let data = user.value;
    let fullUrl = user.value.id
        ? `${baseUrl}/user/store-or-update/${user.value.id}`
        : `${baseUrl}/user/store-or-update`;

    axios
        .post(fullUrl, data)
        .then((response) => {
            loadData(pagination.current_page);
            console.log(response.data.message);
            closeModal();
        })
        .catch(function (error) {
            console.log(error.response);
            closeModal();
        });
};

const destroy = (id) => {
    axios
        .get(`user/destroy/${id}`)
        .then((response) => {
            loadData(1);
        })
        .catch(function (error) {
            console.log(error.response);
        });
};

const closeModal = () => {
    user.value = {
        id: null,
        username: "",
        first_name: "",
        last_name: "",
        email: "",
        mobile_number: "1234567890",
    };
};

onMounted(() => {
    loadData();
});
</script>
