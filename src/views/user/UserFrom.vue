<template>
  <div class="user_from">
    <main-header :title="`${actionLabel}用户`"></main-header>
    <div class="right_page from_main">
      <div class="from_body">
        <el-form
          class="form"
          ref="userForm"
          label-width="100px"
          :model="userClone"
          :rules="rules"
          label-position="left"
        >
          <el-form-item prop="name"  label="姓名">
            <el-input v-model="userClone.name" placeholder="请输入姓名" type="text"></el-input>
          </el-form-item>
          <el-form-item prop="gender" label="选择性别">
            <el-radio-group v-model="userClone.gender">
              <el-radio label="MAN">男</el-radio>
              <el-radio label="WOMAN">女</el-radio>
            </el-radio-group>
          </el-form-item>
          <el-form-item prop="phone" label="手机号">
            <el-input v-model="userClone.phone" placeholder="请输入手机号" type="number"></el-input>
          </el-form-item>
          <el-form-item
            v-if="this.$route.name !== 'user.edit'"
            class="password_box" prop="password"
            label="密码"
          >
            <el-input
              v-model="userClone.password"
              placeholder="请输入密码"
              :type="isSee ? 'text' : 'password'">
            </el-input>
            <icon
              @click.native="isSee = !isSee"
              :type="isSee === true ? 'eye-off' : 'eye'"
              class="pwd_switch"
              color="#999"
              size="24"
            />
          </el-form-item>
          <el-form-item prop="rankCoefficient"  label="职级系数">
            <el-input-number
              v-model="userClone.rankCoefficient"
              :precision="1"
              :step="0.1">
            </el-input-number>
          </el-form-item>
          <el-form-item prop="entryDate"  label="入职时间">
            <el-date-picker
              v-model="userClone.entryDate"
              type="date"
              placeholder="入职时间">
            </el-date-picker>
          </el-form-item>
          <el-form-item prop="roles" label="所属角色">
            <el-select
              v-model="userClone.rolesIds"
              multiple
              filterable
              allow-create
              default-first-option
              placeholder="请选择角色">
              <el-option
                v-for="item in roles"
                :key="item.id"
                :label="item.name"
                :value="item.id">
              </el-option>
            </el-select>
          </el-form-item>
          <el-form-item prop="departmentIds" label="所属部门">
            <el-cascader
              v-model="userClone.departmentIds"
              :props="{
                label: 'name',
                value: 'id',
              }"
              :options="departments"
              filterable
              change-on-select
            ></el-cascader>
          </el-form-item>
          <el-form-item prop="directManager" label="直接经理">
            <el-select
              v-model="userClone.directManager.id"
              filterable placeholder="请选择"
              :filter-method="searchDirectManager"
            >
              <el-option
                v-for="item in directManagerList"
                :key="item.id"
                :label="item.name"
                :value="item.id">
              </el-option>
            </el-select>
          </el-form-item>
          <el-form-item prop="indirectManager" label="间接经理">
            <el-select
              v-model="userClone.indirectManager.id"
              filterable placeholder="请选择间接经理"
              :filter-method="searchIndirectManager"
            >
              <el-option
                v-for="item in indirectManagerList"
                :key="item.id"
                :label="item.name"
                :value="item.id">
              </el-option>
            </el-select>
          </el-form-item>
          <el-form-item>
            <el-button
              @keyup.enter="onSubmit"
              @click="onSubmit"
              :loading="loading"
              type="primary"
            >
              {{actionLabel}}
            </el-button>
            <el-button @click="onCancel">取消</el-button>
          </el-form-item>
        </el-form>
      </div>
    </div>
  </div>
</template>

<script>
import MainHeader from '@/components/MainHeader.vue';
import http from '@/utils/http';
import clone from 'clone';
import Icon from '@/components/Icon.vue';
import { createNamespacedHelpers } from 'vuex';

const { mapState, mapActions } = createNamespacedHelpers('user');

export default {
  name: 'UserFrom',
  components: {
    MainHeader, Icon,
  },
  data() {
    return {
      userClone: this.preprocessUser(clone(this.currentUser)),
      indirectManagerList: [],
      directManagerList: [],
      searchDepartment: [],
      isSee: false,
    };
  },
  beforeRouteUpdate(to, from, next) {
    next();
    this.getUser(this.$route.params.userId);
  },
  computed: {
    ...mapState([
      'currentUser',
    ]),
    actionLabel() {
      return this.$route.name === 'user.create' ? '添加' : '更新';
    },
    loading() {
      return this.$store.state.loading.loadings.user;
    },
    departments() {
      return this.$store.state.department.departmentList;
    },
    roles() {
      return this.$store.state.role.roleList;
    },
    rules() {
      return {
        name: [
          { required: true, message: '请输入商品类型名称', trigger: 'blur' },
          {
            min: 1,
            max: 40,
            message: '长度在 1 到 40 个字符',
            trigger: 'blur',
          },
        ],
        gender: [
          { required: true, message: '请选择性别', trigger: 'blur' },
        ],
        phone: [
          { required: true, message: '请输入手机号', trigger: 'blur' },
        ],
        rankCoefficient: [
          { required: true, message: '请输入职级系数', trigger: 'blur' },
        ],
        directManager: [
          { required: true, message: '请输入直接经理', trigger: 'blur' },
        ],
        indirectManager: [
          { required: true, message: '请输入间接经理', trigger: 'blur' },
        ],
      };
    },
  },
  watch: {
    currentUser() {
      this.userClone = this.preprocessUser(clone(this.currentUser));
    },
    $route() {
      if (this.$route.name === 'user.edit') {
        this.getUser(this.$route.params.userId);
      } else {
        this.userClone = this.preprocessUser({});
      }
    },
  },
  methods: {
    ...mapActions([
      'modifyUser',
      'addUser',
      'getUser',
    ]),
    onSubmit() {
      this.userClone.roles = this.userClone.rolesIds.map(item => ({ id: item }));
      if (this.$route.name === 'user.edit') {
        delete this.userClone.password;
      }
      // delete this.userClone.rolesIds;
      this.$refs.userForm.validate((passed) => {
        if (passed) {
          this[this.userClone.id ? 'modifyUser' : 'addUser'](this.userClone)
            .then(() => {
              this.$emit('success');
              this.$message.success(`${this.actionLabel}成功！`);
              this.$router.push({ name: 'user.index' });
            });
        }
      });
    },
    searchDirectManager(query) {
      this.searchUserList(query, 'directManagerList');
    },
    searchIndirectManager(query) {
      this.searchUserList(query, 'indirectManagerList');
    },
    searchUserList(query, type = 'directManagerList') {
      if (query === '') {
        this[type] = [];
      } else {
        http.get('users/search', {
          params: {
            keyword: query,
            ignoreId: this.$route.params.userId ? this.$route.params.userId : '',
          },
        }).then((res) => {
          this[type] = res.data.data;
        });
      }
    },
    searchDepartmentList(query) {
      if (query === '') {
        this.searchUser = '';
      } else {
        http.get('users/search', {
          params: {
            keyworld: query,
            ignoreId: this.$route.params.userId ? this.$route.params.userId : '',
          },
        }).then((res) => {
          this.searchUser = res.data.data;
        });
      }
    },
    onCancel() {
      this.$emit('canceled');
    },
    preprocessUser(user = {}) {
      const preprocessUser = {
        ...user,
        rolesIds: [],
        department: user.department ? user.department : {},
        directManager: user.directManager ? user.directManager : {},
        indirectManager: user.indirectManager ? user.indirectManager : {},
      };
      if (user.roles) {
        preprocessUser.rolesIds = user.roles.map(item => item.id);
      }
      if (user.directManager) {
        this.directManagerList.push(user.directManager);
      }
      if (user.indirectManager) {
        this.indirectManagerList.push(user.indirectManager);
      }
      return preprocessUser;
    },
  },
  created() {
    this.$store.dispatch('department/getDepartmentList');
    this.$store.dispatch('role/getRoleList');
    if (this.$route.name === 'user.edit') {
      this.getUser(this.$route.params.userId);
    } else {
      this.userClone = this.preprocessUser({});
    }
  },
};
</script>

<style lang="less" scoped>
.from_body{
  background-color: #fff;
  padding: 40px;
  .form{
    width: 500px;
    margin: 0 auto;
  }
  .password_box{
    position: relative;
    .pwd_switch{
      position: absolute;
      right: 10px;
    }
  }
}
</style>
