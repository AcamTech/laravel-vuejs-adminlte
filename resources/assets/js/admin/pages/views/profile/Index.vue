<template>
    <div>
        <section class="content-header">
            <h1>User Profile</h1>
        </section>

        <section class="content">
            <div class="row">
                <div class="col-sm-8 col-md-6 col-lg-4">

                    <!-- Profile Image -->
                    <div class="box box-primary">
                        <div class="box-body box-profile">

                            <profile-image-upload :image="auth_user.profile_image"></profile-image-upload>

                            <h3 class="profile-username text-center">{{ auth_user.name }}</h3>

                            <hr>

                            <div class="form-group" :class="{ 'has-error': errors.has('name') }">
                                <label>Name</label>
                                <input name="name" type="text" class="form-control"
                                    v-model="auth_user.name"
                                    v-validate="'required'"
                                    placeholder="Enter your name">
                            </div>

                            <div class="form-group" :class="{ 'has-error': errors.has('email') }">
                                <label>Email</label>
                                <input name="email" type="text" class="form-control"
                                    v-model="auth_user.email"
                                    v-validate="'required|email'"
                                    placeholder="Enter your email">
                            </div>

                            <div class="form-group" :class="{ 'has-error': errors.has('roles') }">
                                <label>Roles</label>
                                <v-select name="roles"
                                    v-model="auth_user.roles"
                                    :options="available_roles"
                                    label="name"
                                    v-validate="'required'"
                                    placeholder="Select at least one role"
                                    multiple></v-select>
                            </div>

                            <hr>

                            <div class="form-group" :class="{ 'has-error': errors.has('password') }">
                                <label>Password</label>
                                <input name="password" type="password" class="form-control"
                                    v-model="password"
                                    v-validate="password.length > 0 ? 'required|min:6' : ''"
                                    placeholder="Enter your password">
                            </div>

                            <div class="form-group" :class="{ 'has-error': errors.has('password_confirmation') }">
                                <label>Password Repeat</label>
                                <input name="password_confirmation" type="password" class="form-control"
                                    v-model="password_confirmation"
                                    v-validate="password.length > 0 ? 'required|confirmed:password' : ''"
                                    placeholder="Repeat your password">
                            </div>
                        </div>
                        <!-- /.box-body -->

                        <div class="box-footer">
                            <button class="btn btn-primary btn-block" @click="save()"><strong>Save</strong></button>
                        </div>
                    </div>
                    <!-- /.box -->
                </div>
            </div>
        </section>
    </div>
</template>

<script>
import axios from 'axios';
import _ from 'lodash';
import vSelect from 'vue-select';
import ProfileImageUpload from './components/ProfileImageUpload';

export default {
    name: 'ProfilePage',
    components: {
        vSelect, ProfileImageUpload
    },
    data() {
        return {
            available_roles: [],
            password: '',
            password_confirmation: '',
        }
    },
    computed: {
		auth_user() {
            let data = _.clone(this.$store.state.user);
            data.roles = _.clone(this.$store.state.user.roles);

            return data;
		}
    },
    mounted() {
        axios.get(route('admin.api.roles.index'))
            .then((response) => {
                this.available_roles = response.data;
            });
    },
    methods: {
        save() {
            this.$validator.validateAll().then(valid => {
                if (valid) {
                    axios.put(route('admin.api.auth.user.update'), this.getParams())
                        .then((response) => {
                            this.$store.commit('SET_USER', response.data);

                            this.$awn.success('Data was saved successfully');
                        });
                }
            });
        },
        getParams() {
            let params = _.clone(this.auth_user);

            params.roles = _.map(params.roles, (role) => { return role.id });
            params.password = this.password;
            params.password_confirmation = this.password_confirmation;

            return params;
        },
    }
}
</script>