<template>
  <a-modal
    title="New user"
    :width="640"
    :visible="visible"
    :confirmLoading="loading"
    @ok="() => {  $emit('ok') }"
    @cancel="() => { $emit('cancel') }"
  >
    <a-spin :spinning="loading">
      <a-form :form="form" v-bind="formLayout">
        <a-form-item v-show="model && model.id > 0" label="User ID">
          <a-input v-decorator="['id', { initialValue: 0 }]" disabled />
        </a-form-item>
        <a-form-item label="Name">
          <a-input v-decorator="['name', {rules: [{required: false, min: 3, message: 'Name field requires at least 3 characters'}]}]" />
        </a-form-item>
        <a-form-item label="Last Name">
          <a-input v-decorator="['lastname', {rules: [{required: false, min: 5, message: 'Last Name field requires at least 3 characters'}]}]" />
        </a-form-item>
        <a-form-item label="Login">
          <a-input v-decorator="['login', {rules: [{required: false, min: 5, message: 'Login field requires at least 3 characters'}]}]">
            <a-icon slot="prefix" type="user" style="color:rgba(0,0,0,.25)" />
          </a-input> 
        </a-form-item>
        <a-form-item label="Password">
          <a-input type="password" v-decorator="['password', {rules: [{required: false, min: 6, message: 'Password field requires at least 3 characters'}]}]">
            <a-icon slot="prefix" type="lock" style="color:rgba(0,0,0,.25)" />
          </a-input>          
        </a-form-item>
        <a-form-item label="Profile">
          <a-select  v-decorator="['profile',{ rules: [{ required: false, message: 'Please select your gender!' }] },]">
              <a-select-option value="admin">Admin</a-select-option>
              <a-select-option value="user">User</a-select-option>              
            </a-select>
        </a-form-item>
        <!-- <a-form-item v-if="false">
          <a-upload v-decorator="[ 'avatar']">        
        </a-upload>
        </a-form-item> -->
        <a-form-item label="Avatar">                  
          <avatar-modal :img="option.img" ref="modal" @ok="setavatar"/>
        </a-form-item>        
      </a-form>
    </a-spin>
  </a-modal>
</template>

<script>
import pick from 'lodash.pick'
import AvatarModal from './AvatarModal'

// 表单字段
const extraFormFields = ['avatar']

export default {
  components: { 'avatar-modal': AvatarModal },
  props: {
    visible: {
      type: Boolean,
      required: true
    },
    loading: {
      type: Boolean,
      default: () => false
    },
    model: {
      type: Object,
      default: () => null
    }
  },
  data () {
    this.formLayout = {
      labelCol: {
        xs: { span: 24 },
        sm: { span: 7 }
      },
      wrapperCol: {
        xs: { span: 24 },
        sm: { span: 13 }
      }
    }
    return {
      form: this.$form.createForm(this),
      preview: {},
      option: {
        img: '/silhueta.jpg',        
      }
    }
  },
  mounted () {
    console.log('custom modal created')   

    // Registering extra form fields. Fields that isn't as a field decorator
    extraFormFields.forEach(v => this.form.getFieldDecorator(v))    
    
    // Model - Setting values when the form change
    this.$watch('model', () => {
      console.log("model",this.model)
      this.model && this.form.setFieldsValue(pick(this.model, extraFormFields))
    })    
  },
  methods: {
    setavatar (url) {
      this.option.img = url
      this.form.setFieldsValue({avatar:url})
    }    
  }  
}
</script>

<style lang="less" scoped>

  .avatar-upload-wrapper {
    height: 200px;
    width: 100%;
  }

  .ant-upload-preview {
    position: relative;
    margin: 0 auto;
    width: 100%;
    max-width: 180px;
    border-radius: 50%;
    box-shadow: 0 0 4px #ccc;

    .upload-icon {
      position: absolute;
      top: 0;
      right: 10px;
      font-size: 1.4rem;
      padding: 0.5rem;
      background: rgba(222, 221, 221, 0.7);
      border-radius: 50%;
      border: 1px solid rgba(0, 0, 0, 0.2);
    }
    .mask {
      opacity: 0;
      position: absolute;
      background: rgba(0,0,0,0.4);
      cursor: pointer;
      transition: opacity 0.4s;

      &:hover {
        opacity: 1;
      }

      i {
        font-size: 2rem;
        position: absolute;
        top: 50%;
        left: 50%;
        margin-left: -1rem;
        margin-top: -1rem;
        color: #d6d6d6;
      }
    }

    img, .mask {
      width: 100%;
      max-width: 180px;
      height: 100%;
      border-radius: 50%;
      overflow: hidden;
    }
  }
</style>
