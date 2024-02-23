<template>
    <Form @submit="submitContact" :validation-schema="contactFormSchema">
        <div class="form-group">
            <label for="name">Tên</label>
            <Field name="name" type="text" class="form-control" v-model="contactLocal.name" />
            <ErrorMessage name="name" class="error-feedback" />
        </div>
        <div class="form-group">
            <label for="email">E-mail</label>
            <Field name="email" type="email" class="form-control" v-model="contactLocal.email" />
            <ErrorMessage name="email" class="error-feedback" />
        </div>
        <div class="form-group">
            <label for="address">Địa chỉ</label>
            <Field name="address" type="text" class="form-control" v-model="contactLocal.address" />
            <ErrorMessage name="address" class="error-feedback" />
        </div>
        <div class="form-group">
            <label for="phone">Điện thoại</label>
            <Field name="phone" type="tel" class="form-control" v-model="contactLocal.phone" />
            <ErrorMessage name="phone" class="error-feedback" />
        </div>
        <div class="form-group form-check">
            <input name="favorite" type="checkbox" class="form-check-input" v-model="contactLocal.favorite" />
            <label for="favorite" class="form-check-label">
                <strong>Liên hệ yêu thích</strong>
            </label>
        </div>
        <div class="form-group">
            <button v-if="contactLocal._id" type="button" class="btn btn-primary" @click="updateContact()">
                Cập nhật
            </button>
            <button v-else type="submit" class="btn btn-primary">Lưu</button>
            <button v-if="contactLocal._id" type="button" class="ml-2 btn btn-danger" @click="deleteContact()">
                Xóa
            </button>
        </div>
    </Form>
</template>
<script>
import * as yup from "yup";
import { Form, Field, ErrorMessage } from "vee-validate";
import contactService from "@/services/contact.service";
import Swal from 'sweetalert2';
export default {
    components: {
        Form,
        Field,
        ErrorMessage,
    },
    emits: ["submit:contact", "delete:contact"],
    props: {
        contact: { type: Object, required: true }
    },
    data() {
        const contactFormSchema = yup.object().shape({
            name: yup
                .string()
                .required("Tên phải có giá trị.")
                .min(2, "Tên phải ít nhất 2 ký tự.")
                .max(50, "Tên có nhiều nhất 50 ký tự."),
            email: yup
                .string()
                .email("E-mail không đúng.")
                .max(50, "E-mail tối đa 50 ký tự."),
            address: yup.string().max(100, "Địa chỉ tối đa 100 ký tự."),
            phone: yup
                .string()
                .matches(
                    /((09|03|07|08|05)+([0-9]{8})\b)/g,
                    "Số điện thoại không hợp lệ."
                ),
        });
        return {
            // Chúng ta sẽ không muốn hiệu chỉnh props, nên tạo biến cục bộ
            // contactLocal để liên kết với các input trên form
            contactLocal: this.contact ? { ...this.contact } : {
                name: "",
                email: "",
                address: "",
                phone: "",
                favorite: false,
            },
            contactFormSchema,
        };
    },

    methods: {
        async submitContact() {
            this.$emit("submit:contact", this.contactLocal);
            try {
                await contactService.create(this.contactLocal);
                await Swal.fire({
                    title: "Success",
                    text: "Contact created successfully!",
                    icon: "success",
                    timer: 1000, 
                    showConfirmButton: false,
                });
                this.$router.push({ name: 'contactbook' });
            } catch (error) {
                console.error(error);
                Swal.fire({
                    title: "Error",
                    text: "An error occurred while creating the contact",
                    icon: "error",
                    timer: 1000, 
                    showConfirmButton: false,
                });
            }
        },
        async deleteContact() {
            this.$emit("delete:contact", this.contactLocal._id);
            const isConfirmed = await Swal.fire({
                title: 'Bạn muốn xóa Liên hệ này?',
                icon: 'warning',
                showCancelButton: true,
                confirmButtonColor: '#d33',
                cancelButtonColor: '#3085d6',
                confirmButtonText: 'Xóa',
                cancelButtonText: 'Hủy',
            });
            if (isConfirmed.isConfirmed) {
                try {
                    await contactService.delete(this.contactLocal._id);
                    await Swal.fire({
                        title: 'Thành công',
                        text: 'Đã xóa thành công liên hệ này!',
                        icon: 'success',
                        timer: 1000,
                        showConfirmButton: false,
                    });
                    this.$router.push({ name: "contactbook" });
                } catch (error) {
                    console.log(error);
                    Swal.fire({
                        title: 'Lỗi',
                        text: 'Đã xảy ra lỗi khi xóa liên hệ này.',
                        icon: 'error',
                        showConfirmButton: false,
                        timer: 1000,
                    });
                }
            }
        },

        async updateContact() {
            try {
                await contactService.update(this.contactLocal._id, this.contactLocal);
                await Swal.fire({
                    title: 'Success',
                    text: 'Liên hệ được cập nhật thành công.',
                    icon: 'success',
                    timer: 1000,
                    showConfirmButton: false,
                });
                this.$router.push({ name: 'contactbook' });

            } catch (error) {
                console.log(error);
                Swal.fire({
                    title: 'Error',
                    text: 'Có lỗi xảy ra khi cập nhật liên hệ.',
                    icon: 'error',
                    timer: 1000,
                    showConfirmButton: false,
                });
            }
        },
    },
};
</script>
<style scoped>
@import "@/assets/form.css";
</style>