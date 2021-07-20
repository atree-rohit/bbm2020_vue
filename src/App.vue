<style>
	#bbm_logo{
		display: flex;
		justify-content: center;
		align-items: center;
		overflow: hidden
	}
	#bbm_logo img{
		flex-shrink: 0;
		max-width:225px;
	}
	.badge{
		margin: 2px !important;
		font-weight: 100 !important;
		font-size: 0.75em;
		border-radius: 3px !important;
		color: #000 !important;
	}
	.badge-success{
		border: 1px solid rgba(25,135,84,.5);
		background-color: #c3e6cb;
	}
	.badge-warning {
		border: 1px solid rgba(255,193,7,.5);
		background-color: #ffeeba;
	}
	.badge-info {
		border: 1px solid rgba(13,110,253,.5);
		background-color: #bee5eb;
	}
	.rounded-pill {
	    border-radius: 50rem!important;
	}
	.poly_text:hover{
		cursor: pointer;		
	}
	.map-boundary path:hover{
		cursor: pointer;
		fill:rgba(255,255,00,.25);
	}
	#species_table {
		font-size: .8em;
	}
	.states_table tbody tr:hover{
		background-color: yellow;
		cursor: pointer;
	}
	.users-cell{
		cursor: pointer;
		width: 25%;
		background-color: rgba(100,100,255,.125) !important;
	}
	#map-container{
		background-color: black;
	}
	#data-panel{
		max-height:100vh;
		overflow-y: none;
		background-color: #d1e7dd;
	}
</style>
<template>
	<div id="app" class="row">
		<div class="col-xl-6 p-0">
			<div id="map-container" class="svg-container"></div>        	
		</div>
		<div class="col d-flex flex-column" id="data-panel">
			<div id="map_controls" class="border border-success bg-light text-center row m-0">
				<div class="col p-0">
					<table class="table text-center table-sm table-success m-0">
						<thead>
							<tr v-if="selected_state != ''">
								<th colspan=2 class="h2 text-success align-end" v-text="selected_state + ' Data'"></th>
								<th>
									<button class="btn btn-sm btn-danger" @click="select_state('')">Back to All States</button>
								</th>
							</tr>
							<tr class="h1">
								<th v-text="card['taxa']"></th>
								<th v-text="card['individuals']"></th>
								<th v-text="card['users']"></th>
							</tr>
							<tr>
								<th>Unique Taxa</th>
								<th>Individuals</th>
								<th>Observers</th>
							</tr>
						</thead>
					</table>
					<div class="d-flex flex-column" style="background-color: rgb(255, 255, 225);">
						<div class="btn-group my-2 bg-light">
							<button class="btn btn-sm" 
							v-for="btn in Labelbuttons"
							v-text="btn[1]"
							:class="(selected_label == btn[0]) ? 'btn-success': 'btn-outline-primary'"
							@click="selected_label = btn[0]"
							></button>                    	
						</div>
						<div class="btn-group mb-2 bg-light">
							<button class="btn btn-sm mx-1 rounded" 
							:class="selectedProject('counts')"
							@click="selectProject('counts')"
							>Butterfly Counts</button>
							<button class="btn btn-sm mx-1 rounded" 
							:class="selectedProject('inat')"
							@click="selectProject('inat')"
							>iNaturalist</button>
							<button class="btn btn-sm mx-1 rounded" 
							:class="selectedProject('ibp')"
							@click="selectProject('ibp')"
							>India Biodiversity Portal</button>
						</div>                    	
					</div>
				</div>
				<div class="col-3 m-auto" id="bbm_logo">
					<img src="http://diversityindia.org/bbm/Big%20Butterfly%20Month%20Logo.jpg" alt="Big Butterfly Month">
				</div>                
			</div>
			<div style="overflow-y: scroll;" class="border border-info flex-grow-1">
				<table class="table table-sm table-striped bg-light m-0" :class="(selected_state == '')?'states_table':'species_table'">
					<thead class="bg-primary text-light text-center">
						<tr>
							<th v-for="h in header_fields" v-text="h"></th>
						</tr>
					</thead>
					<tbody v-if="selected_state == ''">
						<tr v-for="state in tableData" @click="select_state(state[0])" :class="stateRowColor(state)">
							<td v-text="state[0]"></td>
							<td v-text="state[1]" class="text-center"></td>
							<td v-text="state[2]" class="text-center"></td>
							<td v-text="state[3]" class="text-center"></td>
							<td>
								<span class="badge rounded-pill mx-1" v-for="s in state[4]" :class="'badge-'+portal_colors[s]" v-text="s"></span>
							</td>
						</tr>
					</tbody>
					<tbody v-else class="align-middle">
						<tr v-for="(species, k) in tableData">
							<!-- <td>{{species}}</td> -->
							<td class="text-nowrap" v-text="speciesNames[species[0]][0]"></td>
							<td class="text-nowrap" v-text="speciesNames[species[0]][1]"></td>
							<td v-text="species[2]"></td>
							<td class="users-cell text-center" @click="toggleUserNames(k)">
								<div v-if="selected_species !== k" v-text="noOfUsers(species[1])"></div>
								<!-- badge-user -->
								<template v-else v-for="(s,p) in species[1]">
									<span class="badge rounded-pill" 
										:class="'badge-'+portal_colors[p]"
										v-for="u in s"
										v-text="userNames[p][u]"
										@click.stop="userClick(p,u)"
									></span>
									
								</template>
							</td>
							<td class="text-nowrap">
								<span class="badge rounded-pill"
								v-for="s in species[3]"
								:class="'badge-'+portal_colors[s]"
								v-text="s"
								></span>
							</td>                    		
						</tr>
					</tbody>
				</table>
			</div>
			<div class="text-end text-muted mt-auto ml-auto">
				Created by Rohit George
			</div>
		</div>		
	</div>
  </div>
</template>

<script>
	import country from './data/country.json'
	// import rawData from './data/test_data.json'
	import speciesNames from './data/species.json'
	import userNames from './data/users.json'
	import rawData from './data/data.json'
	import * as d3 from "d3"
	import * as d3Legend from "d3-svg-legend"
	export default {
		name: 'app',
		// props: ["data", "species", "species_ids", "common_names", "users"],
		data () {
			return {
				speciesNames:speciesNames,
				userNames:userNames,
				mapData: country,
				allData:{},
				state_totals:{},
				state_max:{},
				selected_label:"taxa",
				selected_project: ["counts", "inat", "ibp"],
				selected_state:"",
				selected_species:-1,
				tableData:[],
				svgWidth: 0,
				svgHeight: 0,
				svg: null,
				projects: [["all", "All"],["inat", "iNaturalist"],["ibp", "India Biodiveristy Portal"],["counts", "Butterfly Counts"]],
				Labelbuttons:[["taxa", "Unique Taxa"], ["individuals", "Individuals"], ["users", "Observers"]],
				card: {'taxa':0, 'individuals': 0, 'users':0},
				header_fields: [],
				portal_colors: { "inat":"success", "ibp":"warning", "counts":"info"},
			}
		},
		computed:{
			filteredData(){
				var op = JSON.parse(JSON.stringify(this.allData));

				if(this.selected_project.length != 3){
					Object.keys(op).forEach(s => {
						Object.keys(op[s]).forEach(p => {
							let index = this.selected_project.indexOf(p);
							if(index == -1)
								op[s][p] = [];
						});
					});
				}
				return op;
			},
		},
		watch: {
			selected_label: function (val) {
				this.renderMap(); 
			},
			selected_project: function (val) {
				this.renderMap();
			},
		},
		mounted(){
			this.init();
		},
		methods:{
			init(){
				this.svgWidth = window.innerWidth / 2 - 20
				this.svgHeight = window.innerHeight - 10
				if(window.innerWidth < 1140){
					this.svgWidth = window.innerWidth - 50
					this.svgHeight = window.innerHeight/1.25
				}
				this.populateAllData();
				this.renderMap();
			},
			populateAllData(){
				let portals = ["all", "counts", "inat", "ibp"];
				country.features.forEach(state=> {
					let s_name = state.properties.st_nm;
					portals.forEach(p => {
						if(this.state_totals[s_name] == undefined){
							this.state_totals[s_name] = {};
						}
						this.state_totals[s_name][p] = { "users": 0, "taxa": 0, "individuals": 0 };
					})
					if(this.allData[s_name] == undefined)
						this.allData[s_name] = {"counts":[], "inat":[], "ibp": []};
				});

				Object.keys(rawData).forEach(portal => {
					Object.keys(rawData[portal]).forEach(state => {
						Object.values(rawData[portal][state]).forEach(observation => {
							let row = {
									species:observation[0],
									user: observation[1],
									individuals: observation[2]
								};
								this.allData[state][portal].push(row);
						});
					});
				});
			},
			renderMap() {
				this.selected_species = -1;
				this.populateStateTotals();
				this.populateStateMax();
				this.renderTables();

				if (!d3.select("#map-container svg").empty()) {
					d3.selectAll("svg").remove()
				}
				this.svg = d3.select("#map-container").append("svg").attr("preserveAspectRatio", "xMinYMin meet")
					.attr("width", this.svgWidth)
					.attr("height", this.svgHeight)
					.style("background-color", "rgb(190, 229, 235)")
					.classed("svg-content", true)

				var projection = d3.geoMercator().scale(1400).center([85.5, 29.5])
				const path = d3.geoPath().projection(projection)
				const colors = d3.scaleLinear().domain([0, 1, this.state_max[this.selected_label]]).range(["#f77", "#6a8", "#7f9"])
				var legend = d3Legend.legendColor().scale(colors).shapeWidth(55).labelFormat(d3.format(".0f")).orient('horizontal').cells(6)
				let base = this.svg.append("g")
					.classed("map-boundary", true)

				let base_text = base.selectAll("text").append("g")
				base = base.selectAll("path").append("g")

				country.features.forEach(state=> {
					let s_name = state.properties.st_nm
					let shape = base.append("g")
						.data([state])
						.enter().append("path")
						.attr("d", path)
						.attr("stroke", "#333")
						.attr("id", s_name)
						.attr("title", s_name)
						.attr("stroke-width", .5)
						.on("click", (d) => this.select_state(s_name));

					if(s_name == this.selected_state){
						shape.attr("fill", "rgba(255,255,50,1)")
						.attr("stroke", "RED")
					} else if(this.state_totals[s_name] == undefined){
						shape.attr("fill", (d) => colors(-1))
					} else {
						shape.attr("fill", (d) => colors(this.state_totals[s_name]["all"][this.selected_label]))
					}
				})

				country.features.forEach(state=> {
					let s_name = state.properties.st_nm
					let label = base_text.append("g")
						.data([state])
						.enter().append("text")
						.classed("poly_text", true)
						.attr("x", (h) => path.centroid(h)[0] )
						.attr("y", (h) => path.centroid(h)[1] )
						.attr("text-anchor", "middle")
						.attr("font-size",12)
						.text(this.state_totals[s_name]["all"][this.selected_label])
						.on("click", (d) => this.select_state(s_name))
				})

				this.svg.append("g")
					.attr("transform", "translate("+this.svgWidth*.575+", 50)")
					.call(legend)
					.append("text")
					.classed("map_label", true)
					.attr("dx", 5)
					.attr("dy", -10)
					.classed("h1", true)
					.text(this.selected_label.replace(/(?:_| |\b)(\w)/g, function($1){return $1.toUpperCase().replace('_',' ');}))

				let that = this;
				let zoom = d3.zoom()
					.scaleExtent([.5, 7.5])
					.translateExtent([[0,0],[that.svgWidth,that.svgHeight]])
					.on('zoom', function() {
						that.svg.selectAll('.poly_text')
							.attr('transform', d3.event.transform),
						that.svg.selectAll('path')
							.attr('transform', d3.event.transform);
					});
				this.svg.call(zoom);
			},
			populateStateTotals(){
				Object.keys(this.filteredData).forEach(state => {
					let state_taxa = new Set()
					let state_users = new Set()
					let state_individuals = 0
					Object.keys(this.filteredData[state]).forEach(portal => {
						let portal_taxa = new Set()
						let portal_users = new Set()
						let portal_individuals = 0
						Object.values(this.filteredData[state][portal]).forEach(observation => {
							state_taxa.add(observation["species"]);
							state_users.add(observation["user"]);
							state_individuals += observation["individuals"];

							portal_taxa.add(observation["species"]);
							portal_users.add(observation["user"]);
							portal_individuals += observation["individuals"];
						});
						this.state_totals[state][portal]["users"] = portal_users.size;
						this.state_totals[state][portal]["taxa"] = portal_taxa.size;
						this.state_totals[state][portal]["individuals"] = portal_individuals;
					});
					this.state_totals[state]["all"]["users"] = state_users.size;
					this.state_totals[state]["all"]["taxa"] = state_taxa.size;
					this.state_totals[state]["all"]["individuals"] = state_individuals;
				});
			},
			populateStateMax(){
				let cols = ["users", "taxa", "individuals"];
				this.state_max = {
					"users": 0,
					"taxa": 0,
					"individuals": 0,
				}
				Object.values(this.state_totals).forEach(s => {
					cols.forEach(c => {
						if(s.all[c] > this.state_max[c])
							this.state_max[c] = s.all[c];
					})
				});
			},
			renderTables(){
				let table_data = [];
				let portal_names = ["counts", "inat", "ibp"];
				let total_users = 0;
				let total_individuals = 0;

				if(this.selected_state == ""){
					this.header_fields = ["State", "Unique Taxa", "Individuals", "Observers", "Sources"];
					Object.keys(this.state_totals).forEach(state => {
						let portals = [];
						portal_names.forEach(p => {
							if(this.state_totals[state][p].individuals > 0){
								portals.push(p)
							}
						})
						table_data.push([
							state,
							this.state_totals[state].all.taxa,
							this.state_totals[state].all.individuals,
							this.state_totals[state].all.users,
							portals
						]);
					});
					table_data.sort((a, b) => {
						if(a[0] < b[0]) { return -1; }
						if(a[0] > b[0]) { return 1; }
						return 0;
					});
					
					let count = {taxa:new Set(), individuals:0, users:new Set()};

					Object.keys(this.filteredData).forEach(state => {
						Object.keys(this.filteredData[state]).forEach(portal => {
							this.filteredData[state][portal].forEach(observation => {
								count.taxa.add(observation.species);
								count.individuals += observation.individuals;
								count.users.add(observation.user);
								
							})
						});
					});
					this.card = {
						'taxa': count.taxa.size,
						'individuals': count.individuals, 
						'users':count.users.size,
					};

				}
				else {
					let all_species = [];
					let all_u = {"counts":new Set(), "inat":new Set(), "ibp":new Set()};
					this.header_fields = ["Common Name", "Scientific Name", "Individuals", "User", "Source"];
					portal_names.forEach(portal => {
						this.filteredData[this.selected_state][portal].forEach(observation => {
							let new_species_flag = true;
							all_species.forEach((sp, id) => {
								if(sp.scientific_name == observation.species){
									all_u[portal].add(observation.user)
									new_species_flag = false;
									all_species[id]["users"][portal].add(observation.user);
									all_species[id]["individuals"] += observation.individuals;
									all_species[id]["portals"].add(portal);
								}
							});
							
							if(new_species_flag){
								let u = {};
								portal_names.forEach(p => {
									if(p == portal)
										u[p] = new Set([observation.user]);
									else
										u[p] = new Set();
								})
								all_u = u;
								all_species.push({
									scientific_name: observation.species,
									users: u,
									individuals: observation.individuals,
									portals: new Set([portal])
								})
							}
						})
					})

					all_species.forEach(s=>{
						total_individuals += s.individuals;
						table_data.push([
							s.scientific_name,
							s.users,
							s.individuals,
							Array.from(s.portals)
						]);
						// console.log(table_data);
					})

					table_data.sort(function(a, b) {
						return b[3] - a[3];
					})
					this.card = {
						'taxa': table_data.length,
						'individuals': total_individuals, 
						'users':all_u["counts"].size + all_u["inat"].size + all_u["ibp"].size,
					};
				}
				this.tableData = table_data;
			},
			selectedProject(p){
				let index = this.selected_project.indexOf(p);
				let op = "btn-success";

				if(index == -1){
					op = "btn-outline-danger";
				}
				return op;
			},
			toggleUserNames(s){
				if(this.selected_species == s)
					this.selected_species = -1;
				else 
					this.selected_species = s;
			},
			selectProject(p){
				let index = this.selected_project.indexOf(p);
				if (index !== -1) {
					this.selected_project.splice(index, 1);
				} else {
					this.selected_project.push(p)
				}
			},
			stateRowColor(s){
				let op = "";
				if(s[1] + s[2] + s[3] == 0)
					op = "text-danger"
				return op;
			},
			select_state(state){
				this.selected_species = -1;
				if(this.selected_state == state)
					this.selected_state = "";
				else 
					this.selected_state = state;
				this.renderMap();
			},
			getUserPortal(x){
				let portal = "";
				Object.keys(users).forEach(p => {
					users[p].forEach(u => {
						if(x == u){
							portal = p;
						}
					});
				});
				return "badge-" + this.portal_colors[portal];
			},
			getArrayCol(ar, col){
				let op = [];
				ar.forEach(a => {
					op.push(a[col])
				})
				return op;
			},
			noOfUsers(u){
				let op = 0;
				Object.values(u).forEach(portal_users=>{
					op += portal_users.size;
				})
				return op;
			},
			userClick(p,u){
				console.log(p,u)
			}
		}
	}
</script>

