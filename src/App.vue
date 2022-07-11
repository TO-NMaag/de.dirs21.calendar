<template>
  <v-app>
	<div>
      <Calendar/>
	</div>
  </v-app>
</template>

<script>
import Calendar from './components/Calendar';

export default {
  name: 'App',

  components: {
    Calendar,
  },

  data: () => ({
    //
  }),
};
</script>


<style lang="less">

	.test {
		&--test {
			color: green;
		}
	}

</style>