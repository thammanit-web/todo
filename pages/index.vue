<template>
    <v-container>
        <v-row class="mb-12 mt-12 justify-center">
            <v-btn @click="openDialog('add')" class="mx-2 rounded-xl" prepend-icon="mdi-plus" style="font-size: 24px;" large variant="outlined"
                color="primary">Add User</v-btn>
        </v-row>

        <v-data-table-virtual :headers="headers" :items="users" class="rounded-xl"
            :header-props="{ style: 'background: rgba(100, 38, 14, 0.8); color: white;' }"
            style="font-size: 16px; border: 1px solid black; border-collapse: collapse;">
            <template v-slot:item.upload_avatar="{ item }">
                <v-avatar>
                    <v-img :src="'http://localhost:8000/uploads/avatars/' + item.upload_avatar" max-width="50"
                        max-height="50" />
                </v-avatar>
            </template>
            <template v-slot:item.actions="{ item }">
                <v-btn @click="openDialog('edit', item)" class="rounded-lg mx-1" color="primary"
                    variant="outlined">แก้ไข</v-btn>
                <v-btn @click="confirmDelete(item.id)" class="rounded-lg" color="error" variant="outlined">ลบ</v-btn>
            </template>
            <template v-slot:no-data>
                <h1>ไม่มีข้อมูล</h1>
            </template>
        </v-data-table-virtual>

        <v-dialog v-model="dialogVisible" max-width="500px">
            <v-card style="font-size: 16px; border: 1px solid black; border-collapse: collapse;" class="rounded-lg">
                <v-card-title>{{ isEditing ? 'แก้ไข' : 'เพิ่ม' }}</v-card-title>
                <v-card-text>
                    <v-text-field v-model="form.username" label="ชื่อ-นามสกุล" variant="outlined" class="rounded-lg"
                        clearable></v-text-field>
                    <v-text-field v-model="form.email" label="อีเมล" type="email" variant="outlined"
                        clearable></v-text-field>
                    <v-textarea v-model="form.descriptions" label="คำอธิบาย" variant="outlined" clearable></v-textarea>
                    <v-col class="d-flex" align="center" justify="center">
                        <v-img v-if="imagePreview" :src="imagePreview" max-width="100" max-height="100" />
                        <v-file-input v-model="form.upload_avatar" label="รูปภาพ" variant="outlined"
                            prepend-icon="mdi-image-plus" accept="image/*" @change="previewImage">
                        </v-file-input>
                    </v-col>
                </v-card-text>
                <v-card-actions>
                    <v-btn @click="dialogVisible = false" variant="outlined" color="error"
                        class="rounded-lg">ยกเลิก</v-btn>
                    <v-btn @click="saveUser" variant="outlined" color="primary" class="rounded-lg">บันทึก</v-btn>
                </v-card-actions>
            </v-card>
        </v-dialog>


        <v-snackbar v-model="snackbar.visible" :color="snackbar.success ? 'success' : 'error'" timeout="1000">
            {{ snackbar.message }}
        </v-snackbar>

        <v-dialog v-model="deleteDialogVisible" max-width="400px">
            <v-card style="font-size: 16px; border: 1px solid black; border-collapse: collapse;">
                <v-card-title class="mt-2">ยืนยันการลบ?</v-card-title>
                <v-card-actions>
                    <v-btn @click="deleteUserConfirmed" color="error" variant="outlined" class="rounded-lg">ลบ</v-btn>
                    <v-btn @click="deleteDialogVisible = false" color="primary" variant="outlined"
                        class="rounded-lg">ยกเลิก</v-btn>
                </v-card-actions>
            </v-card>
        </v-dialog>

    </v-container>
</template>

<script lang="ts" setup>
import { ref, onMounted } from 'vue'
import axios from 'axios'

const users = ref<any[]>([])
const dialogVisible = ref(false)
const deleteDialogVisible = ref(false)
const isEditing = ref(false)
const form = ref({
    id: null,
    username: '',
    email: '',
    descriptions: '',
    upload_avatar: null
})
const imagePreview = ref()
const snackbar = ref({ visible: false, success: true, message: '' })
const deleteUserId = ref()

const headers = [
    { title: 'ลำดับ', value: 'id', sortable: true },
    { title: 'โปรไฟล์', value: 'upload_avatar', sortable: true },
    { title: 'ชื่อ-นามสกุล', value: 'username', sortable: true },
    { title: 'อีเมล', value: 'email', sortable: true },
    { title: 'คำอธิบาย', value: 'descriptions', sortable: true },
    { title: '', value: 'actions', sortable: false }
]

const fetchUsers = async () => {
    try {
        const response = await axios.get('http://localhost:8000/users')
        users.value = response.data
    } catch (error) {
        console.error('Error fetching users:', error)
    }
}

const openDialog = (type: string, user?: any) => {
    isEditing.value = type === 'edit'
    form.value = user ? { ...user } : { id: null, username: '', email: '', descriptions: '', upload_avatar: null }
    imagePreview.value = user ? 'http://localhost:8000/uploads/avatars/' + user.upload_avatar : null
    dialogVisible.value = true
}

const addUser = async () => {
    try {
        const formData = new FormData()
        formData.append('username', form.value.username)
        formData.append('email', form.value.email)
        formData.append('descriptions', form.value.descriptions)
        if (form.value.upload_avatar) formData.append('upload_avatar', form.value.upload_avatar)
        await axios.post('http://localhost:8000/users', formData)
        fetchUsers()
        dialogVisible.value = false
        showSnackbar('เพิ่มสำเร็จ!', true)
    } catch (error) {
        console.error('Error adding user:', error)
        showSnackbar('Error adding user!', false)
    }
}

const editUser = async () => {
    try {
        const formData = new FormData()
        formData.append('username', form.value.username)
        formData.append('email', form.value.email)
        formData.append('descriptions', form.value.descriptions)
        if (form.value.upload_avatar) formData.append('upload_avatar', form.value.upload_avatar)
        await axios.put(`http://localhost:8000/users/${form.value.id}`, formData)
        fetchUsers()
        dialogVisible.value = false
        showSnackbar('อัพเดตสำเร็จ!', true)
    } catch (error) {
        console.error('Error editing user:', error)
        showSnackbar('Error editing user!', false)
    }
}

const saveUser = () => {
    if (isEditing.value) {
        editUser()
    } else {
        addUser()
    }
}

const confirmDelete = async (id: number) => {
    deleteUserId.value = id
    deleteDialogVisible.value = true
}

const deleteUserConfirmed = async () => {
    if (deleteUserId.value) {
        try {
            await axios.delete(`http://localhost:8000/users/${deleteUserId.value}`)
            fetchUsers()
            deleteDialogVisible.value = false
            showSnackbar('ลบสำเร็จ!', true)
        } catch (error) {
            console.error('Error deleting user:', error)
            showSnackbar('Error deleting user!', false)
        }
    }
}

const showSnackbar = (message: string, success: boolean) => {
    snackbar.value = { visible: true, success, message }
}

const previewImage = () => {
    if (form.value.upload_avatar) {
        const reader = new FileReader()
        reader.onload = (e) => {
            imagePreview.value = e.target?.result as string
        }
        reader.readAsDataURL(form.value.upload_avatar)
    }
}
onMounted(() => {
    fetchUsers()
})
</script>
