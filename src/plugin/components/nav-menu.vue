<script>
  import { Vue, Component, Prop, Watch } from 'vue-property-decorator';
  import _ from 'lodash';

  @Component
  export default class NavMenu extends Vue {
    @Prop(Object) menu;
    @Prop({ type: Function, default: () => true }) filter;
    @Prop(Array) routes;
    @Prop(Boolean) replaceRouter

    active = '';

    getTitle(route) {
      return _.get(route, 'meta.title', '未命名');
    }

    getIcon(route) {
      return _.get(route, 'meta.navbar', '');
    }

    handleSelect(e) {
      this.$emit('select', e);
    }

    renderMenuItem(route) {
      return (
        route && this.filter(route) && (
        <c-permission permission={_.get(route, 'meta.permission')}>
        <el-menu-item
      key={route.name}
      index={route.name}
      data-index={route.name}
        >
        <c-router-link to={{ name: route.name }} replace={this.replaceRouter}>
        <i class={this.getIcon(route)}></i>{this.getTitle(route)}
        </c-router-link>
        </el-menu-item>
        </c-permission>
    )
    )
    }

    renderSubMenu(route, index) {
      // 父级有meta渲染二级路由，父级没有meta当一级路由渲染
      const children = (route.children || []).filter(Boolean);
      return route.meta && children.length ?
    <el-submenu key={route.name} index={String(index)}>
        <template slot="title">
        <div><i class={this.getIcon(route)}></i>{this.getTitle(route)}</div>
      </template>
      {children.map(child => this.renderMenuItem(child))}
    </el-submenu>
    : children.map(child => this.renderMenuItem(child))
    }

    render() {
      return (
        <el-menu class="c-nav-menu" ref="menu" default-active={this.active} props={this.menu}
      onSelect={this.handleSelect}>
        {this.routes.map((route, index) =>
            route && route.children ? this.renderSubMenu(route, index) : this.renderMenuItem(route, index)
          )}
        </el-menu>
    )
    }

    @Watch('$route.name', { immediate: true })
    async onRouteChange() {
      await this.$nextTick();
      const menuEle = this.$refs.menu.$el;

      // 计算当前高亮菜单
      const menuItem = _.get([...menuEle.querySelectorAll('.router-link-active')].pop(), 'parentNode');
      menuItem && (this.active = menuItem.dataset.index);

      // 删除没有子菜单的父菜单
      [...menuEle.querySelectorAll('.el-submenu .el-menu')].forEach(item => {
        if (!item.children.length) {
          menuEle.removeChild(item.parentNode);
        }
      });
    }
  }
</script>
