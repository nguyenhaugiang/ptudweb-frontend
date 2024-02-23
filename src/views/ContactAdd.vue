<template>
    <div class="page justify-content-center align-items-center">
        <h4>Them moi lien he</h4>
        <ContactForm />
    </div>
</template>
<script>
import ContactForm from '@/components/ContactForm.vue';
import Swal from 'sweetalert2';

export default {
    components: {
        ContactForm,
    },
    
    created() {
        let timerInterval = 500;
        Swal.fire({
            title: 'Loading...',
            timer: 500,
            timerProgressBar: true,
            didOpen: () => {
                Swal.showLoading();
            },
            willClose: () => {
                clearInterval(timerInterval);
            },
        });
    },
}
</script>