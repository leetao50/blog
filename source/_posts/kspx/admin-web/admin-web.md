---
title: admin-web
date: 2024-07-08 10:50:00
tags:
---

# app

## ngOnInit

```javascript

ngOnInit() {
    // 初始化导航链接
    this.headerNavs = [];
    initHeaderNavs()
    /*{
        //在本地缓存中获取用户信息，如果没有获取到 直接退出；
        //如果获取到了，根据用户权限 设置 菜单
        if (authorities.find(authority => authority.authority === AuthorityType.ROLE_PLATFORM_ADMIN)) {
        this.headerNavs = platformAdminNavs;
        } else if (authorities.find(authority => authority.authority === AuthorityType.ROLE_CONTENT_ADMIN)) {
        this.headerNavs = contentAdminNavs;
        }
    */}
    //订阅 路由
    {
        // 更新登录状态
        this.isLogin = this.authService.authenticate();
    }
    //更新当前选中菜单
    this.updateActiveNav();

```
## 登录
```htm
<button class="nec-btn-primary" routerLink="/login" type="button">登录</button>
```

## 修改密码
```html
<li class="menu" (click)="goToPage('/user/password/update')">
```

## 退出登录
```javascript
logout(){
    // 清理相关数据
    this.showDropdown = false;
    //         * 用户登出, 清空token信息
    this.authService.logout();
    {
        this.ls.del('token');
        this.errorCollectService.clear();
        this.ls.del('kyHCloudLoginParam');
    }
    //         * 删除当前用户的信息, 常用于用户退出登录时注销信息
    this.ls.del(this.LS_USER_INFO);
    this.isLogin = false;
    this.headerNavs.forEach(menu => (menu.active = false));

    this.router.navigate(['/login']);
}
```

# login NecLoginComponent

## ngOnInit
```javascript

ngOnInit() {
    //获取本地缓存认证信息
    if (this.authService.authenticate()) {
      this.router.navigate(['/home']);
    }
    // 是否是admin端登录
    this.isAdminProject = NecProject.NAME === 'admin';
  }

```
## 登录 login()

```javascript

    /*
        1.显示蒙版；

        2.用户登录 
        this.authService.login(this.username, this.password)
        url = NecUrl.SERVER_URL + "/auth/oauth/token"
        登录成功后，本地保存认证信息 
        this.ls.set('token', token);

        3.设置用户权限

        4.获取user详细信息
        getUserInfoFromServer = function (allowAuthorities, url)
        if (url === void 0) { url = NecUrl.SERVER_URL + "/server/me"; }
        设置当前用户的信息, 常用于登录时获取用户信息后进行保存
        this.ls.set(this.LS_USER_INFO, userInfo);

        4.1 获取当前用户的角色
        4.2 获取用户职位信息
        getUserPositionInfo = function (userId, type)
        url = NecUrl.SERVER_URL + '/server/userAndPositions/' + userId + '?type=admin';

    */
  login() {
    this.loadingService.show();
    this.authService.login(this.username, this.password).subscribe(
        /*
        {
            if (url === void 0) { url = NecUrl.SERVER_URL + "/auth/oauth/token"; }
            var postBody = "username=" + encodeURIComponent(username) + "&password=" + encodeURIComponent(btoa(password)) + "&grant_type=password&isBase64Password=true";
            return this.http.post(url, postBody, {
                headers: new http.HttpHeaders({
                    Authorization: 'Basic ' + btoa('web:'),
                    'Content-Type': 'application/x-www-form-urlencoded; charset=UTF-8'
                })
            });
        }
        */
      token => {
        //本地保存认证信息 this.ls.set('token', token);
        this.authService.authenticate(token);

        // 当前有权限的角色
        let authorityTypes = [];
        if (this.isAdminProject) {
          authorityTypes = [AuthorityType.ROLE_CONTENT_ADMIN, AuthorityType.ROLE_PLATFORM_ADMIN];
        } else {
          authorityTypes = [
            AuthorityType.ROLE_DEPT_ADMIN,
            AuthorityType.ROLE_USER,
            AuthorityType.ROLE_TENANT,
            AuthorityType.ROLE_NEW_NURSE,
            AuthorityType.ROLE_TENANT
          ];
        }
        // 获取user
        this.userService.getUserInfoFromServer(authorityTypes).subscribe(
            /**
             * 
            getUserInfoFromServer = function (allowAuthorities, url) {
                var _this = this;
                if (url === void 0) { url = NecUrl.SERVER_URL + "/server/me"; }
                * 查询当前登录用户信息
                return this.userInterface.getUserInfo(url).pipe(operators.map(function (result) {
                    * 
                        NecUserInterface.prototype.getUserInfo = function (url) {
                            if (url === void 0) { url = NecUrl.SERVER_URL + "/server/me"; }
                            return this.http.get(url);
                        };
                    *
                    var userAuthority = _this.getAuthority(result);
                    if (!allowAuthorities || (allowAuthorities && allowAuthorities.indexOf(userAuthority.toLocaleString()) > -1)) {
                        return result;
                    }
                    else {
                        throw new Error('用户名或密码错误');
                    }
                }));
            */
          user => {

            // * 设置当前用户的信息, 常用于登录时获取用户信息后进行保存
            // this.ls.set(this.LS_USER_INFO, userInfo);
            this.userService.setUserInfo(user);

            //  * 获取当前用户的角色
            let authority = this.userService.getAuthority();
            /**
             * getAuthority (user) {
                    var userInfo;
                    if (user) {
                        userInfo = user;
                    }
                    else {
                        userInfo = this.getUserInfo();
                    }
                    if (userInfo && userInfo.account && userInfo.account.authorities && userInfo.account.authorities[0]) {
                        for (var i = 0; i < userInfo.account.authorities.length; i++) {
                            var isManager = this.getIsManager();
                            if (userInfo.account.authorities[i].authority === exports.AuthorityType.ROLE_USER) {
                                if (isManager) {
                                    return exports.AuthorityType.ROLE_DEPT_ADMIN;
                                }
                                else {
                                    return exports.AuthorityType.ROLE_USER;
                                }
                            }
                        }
                        return userInfo.account.authorities[0].authority;
                    }
                    else {
                        return null;
                    }
                };
            */
            // 获取普通用户头像
            if (
              authority === AuthorityType.ROLE_DEPT_ADMIN ||
              authority === AuthorityType.ROLE_USER ||
              authority === AuthorityType.ROLE_TRAINEE ||
              authority === AuthorityType.ROLE_NEW_NURSE
            ) {
                //         * 获取用户信息及职位信息（获取头像url也调用这个)
              this.userService.getUserPositionInfo(user.me.id).subscribe(result => {});
              /*
                getUserPositionInfo = function (userId, type) {
                    var url;
                    if (type === 'admin') {
                        url = NecUrl.SERVER_URL + '/server/userAndPositions/' + userId + '?type=admin';
                    }
                    else {
                        url = NecUrl.SERVER_URL + '/server/userAndPositions?type=user';
                    }
                    return this.http.get(url);
                };
              */
            }
            this.loadingService.hide();
            this.router.navigate(['/home']);
          },
          error => {
            this.toast.warning(error.message);
            this.loadingService.hide();
          }
        );
      },
      error => {
        this.toast.warning('用户名或密码错误');
        this.loadingService.hide();
      }
    );
  }

```

## 服务
```javascript

//用户登录认证
url = auth + "/auth/oauth/token"
table:
    account
    authority


//获取当前用户信息
url = server + "/server/me"
table:
    //医院管理员登录
    hospital->tenant
    dept
    tenant_group

    //平台管理员登录
    platform_admin

    //内容管理员登录
    content_admin

    //普通用户登录
    user_position   //用户部门信息
    reg_user        //用户信息


```

# institution/tenant/list 机构管理->医院管理

## ngOnInit

```javascript
//获取租户列表
getTenantList(pageNo: number, perPageCount: number, tenantSearchParam: TenantSearchParam): Observable<PageResult>

let url = NecUrl.SERVER_URL + '/server/tenants';

return this.http.get<PageResult>(url, { params: params });


@GetMapping("/tenants")@ResponseStatus(HttpStatus.OK)public Page<TenantDTO> queryTenants(@Valid TenantFindDTO findDTO,                                    @PageableDefault(sort = {"createTime"}, direction = Sort.Direction.DESC) Pageable pageable) 

table:
    hospital、tenant



```

## 新增医院
```javascript
    //导航到新增界面
    this.router.navigate(['/institution/tenant/create']);
    
    //在新增界面 save() 方法中

    //* 新增租户
    let url = NecUrl.SERVER_URL + '/server/tenants';
    return this.http.post<Number>(url, tenantCreateDTO);

    // 服务端方法
    @PostMapping("/tenants")
    @Transactional@ResponseStatus(HttpStatus.CREATED)
    public Long create(@RequestBody @Valid TenantDTO tenantDTO)
        1. 在 auth库判断账号是否存在

            necAuthRestTemplate.getForObject("/accounts/existence?username={username}",Boolean.class,username)
            
            Table:
                auth:account
        
        2. 在 auth库 添加账号
            
            necAuthRestTemplate.postForObject("/accounts", new AccountDTO(account),Long.class);

            Table:
                auth:account //添加到表中

            //如果用户已存在id，还需要注销 token 
            if(account.getId()!=null){    tokenRevokeService.revoke(account.getUsername());}

        3. 在 server库 添加账号
            public Long addTenant(Account account, Tenant tenant) 
            {    
                tenant.initAddTenant();    
                Tenant saveTenant = tenantRepository.save(tenant);    
                Long tenantId = saveTenant.getId();    
                aclManager.definePermission(new ObjectIdentityImpl(Tenant.class, tenantId), null, Collections.singletonList(account.getUsername()), BasePermission.ADMINISTRATION);    
                //广播新建医院事件    
                publisher.publishEvent(new TenantAddEvent(tenantId));    
                tenantChangeProducer.sendTenantAddMessage(tenantId);    
                return tenantId;
            }

            Tbale
                server: tenant
                tenant.configId->tenant_config.id
                tenant.syncConfigId->tenant_sync_config.id
                hospital


        4. 发生异常 删除 auth库添加账号信息
            necAuthRestTemplate.delete("/accounts/{accountId}",accountId);



    //更新租户
    let url = NecUrl.SERVER_URL + '/server/tenants/' + updateTenant.tenant.id;

    return this.http.put(url, updateTenant);

```

## 点击列表行元素
```javascript
    this.router.navigate(['/institution/tenant/detail', tenantInfo.tenant.id]);
    //重定向到 '/institution/tenant/create'

```

## 点击列表-使用状态

```javascript
//更新租户状态
toggleEnable
    let url = NecUrl.SERVER_URL + '/server/tenants/' + tenantId;
    return this.http.patch(url, { enable: enable });

```

## 点击列表行-护理管理对接
```javascript
toggleOpenClient

    if (tenant.syncConfig.openYdylClient) {
        // 关闭护理管理对接
        // 删除client
        let url = NecUrl.SERVER_URL + '/auth/integrationClients/' + tenantId;
        return this.http.delete(url);
      } else {
        // 开启
        this.openClientModal(tenant.id);
      }
```

## 点击列表行-编辑
```javascript
modifyClient
    //如果为尚未对接护理管理，需要先对接
    if (!tenant.syncConfig.openYdylClient) {
      this.toast.warning('该医院为未对接状态，请先更改对接状态');
      return;
    }
    this.curTenant = tenant;
    this.openClientModal(tenant.id); 
```

## 弹窗修改框 openClientModal - modify

```javascript
modify
    //新增client
    let url = NecUrl.SERVER_URL + '/auth/integrationClients/';
    return this.http.post<IntegrationClient>(url, integrationClient);

    //同步server端client 状态
    let url = NecUrl.SERVER_URL + '/server/tenants/' + tenantId + '/clients';
    let params = new HttpParams().set('openClient', openClient.toString());
    return this.http.put(url, params);

```


# institution/school/list  机构管理->学校管理

## ngOnInit

```javascript

    let url = NecUrl.SERVER_URL + '/trnee/schools';

    table:
        school
```

## 新增学校

```javascript
this.router.navigate(['/institution/school/create']);

    let url = NecUrl.SERVER_URL + '/trnee/schools/' + schoolId;
    return this.http.get<SchoolInfo>(url);

```

# institution//account/list 机构管理->账号管理

## ngOnInit

```javascript

    let url = NecUrl.SERVER_URL + '/server/' + (isPlatformAdmin ? 'platformAdmins' : 'contentAdmins');

    return this.http.get<PageResult>(url, { params: params });

    table:
        content_admin
        platform_admin
        auth.account
        auth.authority
```


# analysis/register-user/list 业务统计->注册用户一览

## ngOnInit


reg_user



