<template>
  <nav class="navbar navbar-expand-lg fixed-top navbar-dark" :class="{
    'bg-blue': nodeSelectLabel.match(/Main/),
    'bg-success': nodeSelectLabel.match(/Test/),
    'bg-info': nodeSelectLabel.match(/Hooks|custom-node/)
  }" aria-label="Main navigation">
    <div class="container-fluid">
      <router-link class="nes nav navbar-brand" to="/">
        <span class="d-block d-md-none">XRPL <small>Explorer</small></span>
        <span class="d-none d-md-block">XRP Ledger Explorer</span>
      </router-link>
      <button class="navbar-toggler p-0 border-0" type="button" @click="navbarCollapsed = !navbarCollapsed"
        aria-label="Toggle navigation">
        <span class="navbar-toggler-icon"></span>
      </button>

      <div class="navbar-collapse offcanvas-collapse" :class="{ open: navbarCollapsed }">
        <ul class="navbar-nav me-auto mb-2 mb-lg-0">
          <li class="nav-item dropdown">
            <a class="nav-link dropdown-toggle" href="#" id="dropdown01" data-bs-toggle="dropdown"
              aria-expanded="false">{{ nodeSelectLabel }}</a>
            <ul class="dropdown-menu shadow" aria-labelledby="dropdown01">
              <li><a class="dropdown-item" href="https://explorer.xrplf.org"><b>Mainnet</b></a></li>
              <li><a class="dropdown-item" href="https://explorer-testnet.xrplf.org">Testnet</a></li>
              <li><a class="dropdown-item" href="https://hooks-testnet-v3-explorer.xrpl-labs.com">Hooks Testnet V3</a>
              </li>
              <li><a class="dropdown-item" href="https://hooks-testnet-v2-explorer.xrpl-labs.com">Hooks Testnet V2</a>
              </li>
            </ul>
          </li>

          <li class="nav-item dropdown">
            <a class="nav-link dropdown-toggle" href="#" id="dropdown01" data-bs-toggle="dropdown"
              aria-expanded="false">Others</a>
            <ul class="dropdown-menu shadow" aria-labelledby="dropdown01">
              <li>
                <a class="dropdown-item" style="white-space: nowrap;"
                  href="https://github.com/XRPLF/XRPL-Technical-Explorer" target="_blank">
                  <i class="fab fa-github-square"></i><span class="ps-2">Source</span>
                </a>
              </li>
              <li>
                <router-link  class="dropdown-item" to="/command">
                  <i class="fas fa-code"></i><span class="ps-2">Command</span>
                </router-link>
              </li>
              <li>
                <router-link  class="dropdown-item" to="/analytic">
                  <i class="fas fa-chart-pie"></i><span class="ps-2">Analytic</span>
                </router-link>
              </li>
            </ul>
          </li>
        </ul>
        <form class="d-flex" @submit="search">
          <input v-model="query" class="form-control border border-2 border-dark py-0 me-2" type="search"
            placeholder="Search" aria-label="Search">
          <!-- <button class="btn btn-outline-success" type="submit">Search</button> -->
          <button :disabled="!validQuery" type="submit" :class="{ 'is-success': validQuery, 'is-disabled': !validQuery }"
            class="py-0 px-2 nes-btn">Search</button>
        </form>
      </div>
    </div>
  </nav>
</template>

<script>
export default {
  name: 'Header',
  data () {
    return {
      navbarCollapsed: false,
      query: ''
    }
  },
  computed: {
    nodeSelectLabel () {
      if (this.$net.test) {
        return 'Testnet (Change)'
      }
      if (this.$net.hooks) {
        return 'Hooks (Change)'
      }
      return 'Mainnet (Change)'
    },
    validQuery () {
      const commands = this.$router.options.routes.filter(r => {
        return r?.meta?.isPublicCommand && r.name.slice(0, 1) !== '_'
      }).map(r => r.name.split('_').slice(1).join('_'))

      const query = this.query.trim()

      if (query.length < 2) {
        return false
      }

      if (query.match(/^r[a-zA-Z0-9]{15,}/)) {
        // XRPL account address
        return query
      }
      if (query.match(/^[a-fA-F0-9]{64}/)) {
        // Ledger / TX / Object hash
        return query
      }
      if (query.match(/^[0-9]{5,}/) && Number(query) >= 32570) {
        // Ledger Index
        return query
      }
      const possibleCommands = commands.filter(c => c.match(query.toLowerCase()))
      if (possibleCommands.length > 0) {
        return possibleCommands
      }

      return false
    }
  },
  components: {
  },
  methods: {
    search (e) {
      e.preventDefault()
      let navTo
      const navQuery = {}

      if (this.validQuery && typeof this.validQuery === 'string') {
        navTo = '/' + this.validQuery
      }
      if (this.validQuery && Array.isArray(this.validQuery) && this.validQuery.length > 0) {
        if (this.validQuery.length === 1) {
          navTo = '/' + this.validQuery[0]
        }
        if (this.validQuery.length > 1) {
          console.log(this.validQuery)
          navTo = '/command'
          // console.log(this.validQuery)
          Object.assign(navQuery, {
            c: this.validQuery
          })
        }
      }
      console.log(navTo)
      if (
        navTo &&
        (
          this.$route.path !== navTo ||
          JSON.stringify(navQuery) !== JSON.stringify(this?.$route?.query || {})
        ) &&
        !(
          JSON.stringify(navQuery) === JSON.stringify(this?.$route?.query || {}) &&
          this.$route.path === navTo
        )
      ) {
        this.$router[Object.keys(navQuery).length > 0 ? 'replace' : 'push']({
          path: navTo,
          query: navQuery
        })
      }
      return false
    }
  },
  mounted () {
  }
}
</script>

<style lang="scss" scoped></style>
