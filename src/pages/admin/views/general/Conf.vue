<template>
  <div class="view">
    <Panel :title="$t('m.SMTP_Config')">
      <el-form label-position="left" label-width="70px" :model="smtp">
        <el-row :gutter="20">
          <el-col :span="12">
            <el-form-item :label="$t('m.Server')" required>
              <el-input v-model="smtp.server" placeholder="SMTP Server Address"></el-input>
            </el-form-item>
          </el-col>
          <el-col :span="12">
            <el-form-item :label="$t('m.Port')" required>
              <el-input type="number" v-model="smtp.port" placeholder="SMTP Server Port"></el-input>
            </el-form-item>
          </el-col>
          <el-col :span="12">
            <el-form-item :label="$t('m.Email')" required>
              <el-input v-model="smtp.email" placeholder="Account Used To Send Email"></el-input>
            </el-form-item>
          </el-col>
          <el-col :span="12">
            <el-form-item :label="$t('m.Password')" label-width="90px" required>
              <el-input v-model="smtp.password" type="password" placeholder="SMTP Server Password"></el-input>
            </el-form-item>
          </el-col>
          <el-col :span="24">
            <el-form-item label="TLS">
              <el-switch
                v-model="smtp.tls">
              </el-switch>
            </el-form-item>
          </el-col>
        </el-row>
      </el-form>
      <el-button type="primary" @click="saveSMTPConfig">Save</el-button>
      <el-button type="warning" @click="testSMTPConfig"
                 v-if="saved" :loading="loadingBtnTest">Send Test Email</el-button>
    </Panel>

    <Panel :title="$t('m.Website_Config')">
      <el-form label-position="left" label-width="100px" ref="form" :model="websiteConfig">
        <el-row :gutter="20">
          <el-col :span="8">
            <el-form-item :label="$t('m.Base_Url')" required>
              <el-input v-model="websiteConfig.website_base_url" placeholder="Website Base Url"></el-input>
            </el-form-item>
          </el-col>
          <el-col :span="8">
            <el-form-item :label="$t('m.Name')" required>
              <el-input v-model="websiteConfig.website_name" placeholder="Website Name"></el-input>
            </el-form-item>
          </el-col>
          <el-col :span="8">
            <el-form-item :label="$t('m.Shortcut')" required>
              <el-input v-model="websiteConfig.website_name_shortcut" placeholder="Website Name Shortcut"></el-input>
            </el-form-item>
          </el-col>
          <el-col :span="24">
            <el-form-item :label="$t('m.Footer')" required>
              <el-input type="textarea" :autosize="{ minRows: 2, maxRows: 4}" v-model="websiteConfig.website_footer"
                        placeholder="Website Footer HTML"></el-input>
            </el-form-item>
          </el-col>
          <el-col :span="24">
            <el-col :span="12">
              <el-form-item :label="$t('m.Allow_Register')" label-width="200px">
                <el-switch
                  v-model="websiteConfig.allow_register"
                  active-color="#13ce66"
                  inactive-color="#ff4949">
                </el-switch>
              </el-form-item>
            </el-col>
            <el-col :span="12">
              <el-form-item :label="$t('m.Submission_List_Show_All')" label-width="200px">
                <el-switch
                  v-model="websiteConfig.submission_list_show_all"
                  active-color="#13ce66"
                  inactive-color="#ff4949">
                </el-switch>
              </el-form-item>
            </el-col>
          </el-col>
        </el-row>

        <div class="extra-menu__header">{{$t('m.Website_ExtraMenu')}}</div>
        <el-row :gutter="20" class="extra-menu__body">
          <el-col :span="24">
            <el-form label-position="right" label-width="75px">
              <el-col :span="24" v-for="(item,index) in websiteConfig.extra_menu" :key="index">
                <el-col :span="7">
                  <el-form-item :label="$t('m.Website_ExtraMenu_Name')" >
                    <el-input v-model="item.name" />
                  </el-form-item>
                </el-col>
                <el-col :span="7">
                  <el-form-item :label="$t('m.Website_ExtraMenu_Url')" >
                    <el-input v-model="item.url" />
                  </el-form-item>
                </el-col>
                <el-col :span="5.5">
                  <el-form-item :label="$t('m.Website_ExtraMenu_Icon')">
                    <el-input v-model="item.icon" />
                  </el-form-item>
                </el-col>
                <el-col :span="3.5">
                  <el-form-item>
                    <el-button type="text" class="menu_del_btn" @click="handleDelMenu(item)">{{$t('m.Del_Menu')}}</el-button>
                  </el-form-item>
                </el-col>
              </el-col>
              <el-col :span="24">
                <el-button plain :style="{'width':'100%'}" @click="handleAddExtraEmnu">{{$t('m.Add_Menu')}}</el-button>
              </el-col>
            </el-form>
          </el-col>
        </el-row>
      </el-form>
      <save @click.native="saveWebsiteConfig"></save>
    </Panel>
  </div>
</template>

<script>
  import api from '../../api.js'

  export default {
    name: 'Conf',
    data () {
      return {
        init: false,
        saved: false,
        loadingBtnTest: false,
        smtp: {
          server: 'smtp.example.com',
          port: 25,
          password: '',
          email: 'email@example.com',
          tls: true
        },
        websiteConfig: {
          extra_menu: []
        }
      }
    },
    mounted () {
      api.getSMTPConfig().then(res => {
        if (res.data.data) {
          this.smtp = res.data.data
        } else {
          this.init = true
          this.$warning('Please setup SMTP config at first')
        }
      })
      api.getWebsiteConfig().then(res => {
        this.websiteConfig = res.data.data
      }).catch(() => {
      })
    },
    methods: {
      saveSMTPConfig () {
        if (!this.init) {
          api.editSMTPConfig(this.smtp).then(() => {
            this.saved = true
          }, () => {
          })
        } else {
          api.createSMTPConfig(this.smtp).then(() => {
            this.saved = true
          }, () => {
          })
        }
      },
      testSMTPConfig () {
        this.$prompt('Please input your email', '', {
          inputPattern: /[\w!#$%&'*+/=?^_`{|}~-]+(?:\.[\w!#$%&'*+/=?^_`{|}~-]+)*@(?:[\w](?:[\w-]*[\w])?\.)+[\w](?:[\w-]*[\w])?/,
          inputErrorMessage: 'Error email format'
        }).then(({value}) => {
          this.loadingBtnTest = true
          api.testSMTPConfig(value).then(() => {
            this.loadingBtnTest = false
          }, () => {
            this.loadingBtnTest = false
          })
        }).catch(() => {
        })
      },
      saveWebsiteConfig () {
        api.editWebsiteConfig(this.websiteConfig).then(() => {
        }).catch(() => {
        })
      },
      handleAddExtraEmnu () {
        this.websiteConfig.extra_menu.push({name: '', url: '', icon: ''})
      },
      handleDelMenu (row) {
        const index = this.websiteConfig.extra_menu.indexOf(row)
        if (index > -1) {
          this.websiteConfig.extra_menu.splice(index, 1)
        }
      }
    }
  }
</script>

<style lang="less" scoped>
  .extra-menu__header {
    margin-top: 0px;
    margin-right: 0px;
    margin-bottom: 0px;
    margin-left: -15px;
    color: #333;
    border-color: #ddd;
    font-size: 18px;
    font-weight: 300;
    letter-spacing: 0.025em;
    height: 60px;
    line-height: 45px;
    padding: 10px 15px;
    border-bottom: 1px solid #eee;
    border-top-left-radius: 3px;
    border-top-right-radius: 3px;
  }
  .extra-menu__body {
    padding-top: 15px;
    padding-bottom: 15px;
  }
  .menu_del_btn {
    color: #F56C6C;
  }
</style>
