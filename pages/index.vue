<template>
	<div class="container">
		<Notification
			v-on:displaynot="active = false"
			v-if="active"
			:type="errorType"
			:message="errorMessage"
		/>
		<form>
			<div class="field">
				<label class="label">Initial Details: </label>

				<div class="field columns">
					<div class="field column is-2">
						<label class="label is-small">Orientation: </label>
						<div class="select">
							<select v-model="iniPos" required>
								<option value="N">N</option>
								<option value="S">S</option>
								<option value="W">W</option>
								<option value="E">E</option>
							</select>
						</div>
					</div>

					<div class="field column">
						<label class="label is-small">Coordinate X</label>
						<div class="control">
							<input
								class="input spacing"
								type="number"
								v-model.number="inCoorX"
								placeholder="X"
								required
							/>
						</div>
						<p class="help">eg. 20</p>
					</div>

					<div class="field column">
						<label class="label is-small">Coordinate Y</label>
						<div class="control">
							<input
								class="input"
								type="number"
								v-model.number="inCoorY"
								placeholder="Y"
								required
							/>
						</div>
						<p class="help">eg. 20</p>
					</div>
				</div>
			</div>

			<div class="field">
				<label class="label">Size of square:</label>
				<div class="control">
					<input
						class="input"
						type="number"
						v-model="cooX"
						placeholder="Width"
						required
					/>
				</div>
				<p class="help">eg. 20</p>
			</div>

			<div class="field">
				<div class="control">
					<input
						class="input"
						type="number"
						v-model="cooY"
						placeholder="Height"
						required
					/>
				</div>
				<p class="help">eg. 30</p>
			</div>

			<div class="field">
				<label class="label">Commands:</label>
				<div class="field">
					<div class="control">
						<input class="input" @keyup="move" type="text" v-model="commands" />
					</div>
					<p class="help">eg. lrlalla</p>
				</div>
			</div>
		</form>

		<div class="card">
			<div class="card-content">
				<h2 class="title is-4">Current Rover Information:</h2>

				<h3 class="subtitle">
					Position (x,y):
					<!--adds class red if out of the bounds-->
					<span :class="!withinBounds && 'red'"
						>({{ coordinates.x != null ? coordinates.x : inCoorX || 0 }},{{
							coordinates.y != null ? coordinates.y : inCoorY || 0
						}})</span
					>
				</h3>
				<h3 class="subtitle">
					Orientation:
					{{ lastCommand ? lastCommand : iniPos || 0 }}
				</h3>
				<h3 class="subtitle">
					Within Bounds:
					<span :class="!withinBounds && 'red'">
						<!--adds class red if out of the bounds-->
						{{ withinBounds ? 'YES' : 'NO' }}
					</span>
				</h3>
			</div>
		</div>
	</div>
</template>

<script>
	import Notification from '../components/Notification'
	export default {
		components: { Notification },
		data() {
			return {
				iniPos: null,
				inCoorX: null,
				inCoorY: null,
				cooY: null,
				cooX: null,
				commands: '',
				coordinates: {},
				lastCommand: null,
				withinBounds: true,
				active: true,
				defaultMsg:
					'Please note that coordinates begins "0" and square size begin "1", it means eg. coordinate (2,0) is outside the square size "2x2"',
				errorMessage: '',
				errorType: 'is-warning',
			}
		},
		methods: {
			move() {
				let commandsArr = this.commands.toUpperCase().split('')

				if (
					//Validation if command is valid. Accpets only L,R,A
					this.validateCommands(commandsArr) &&
					//Validation if square size and initial coordinates are valid. Accpets only positive, whole numbers
					this.validateCoo(this.inCoorX) &&
					this.validateCoo(this.inCoorY) &&
					this.validateCoo(this.cooY) &&
					this.validateCoo(this.cooX)
				) {
					this.errorType = 'is-warning'
					this.errorMessage = this.defaultMsg
					this.active = true

					this.coordinates = { x: this.inCoorX, y: this.inCoorY } // always sets initial coordinates as starting point
					this.lastCommand = this.iniPos

					// checks the current orientation and changes current orientation to new one.
					//"lastCommand" is always current orientation. After to new command set it is overwritted with new current.
					commandsArr.forEach((command) => {
						let moveDir = command + this.lastCommand
						switch (moveDir) {
							// if the new command "L" or "R" is set it only chnges the current orientation depending what orientation it previously was.
							case 'LN':
							case 'RS':
								this.lastCommand = 'W'
								break
							// if there is command "A" additionally change the coordinate.
							case 'AW':
								this.coordinates.x--
								this.lastCommand = 'W'
								break
							case 'RN':
							case 'LS':
								this.lastCommand = 'E'
								break
							case 'AE':
								this.coordinates.x++
								this.lastCommand = 'E'
								break
							case 'RW':
							case 'LE':
								this.lastCommand = 'N'
								break
							case 'AN':
								this.coordinates.y++
								this.lastCommand = 'N'
								break
							case 'RE':
							case 'LW':
								this.lastCommand = 'S'
								break
							case 'AS':
								this.coordinates.y--
								this.lastCommand = 'S'
								break
						}
					})

					//checks if within bounds
					this.withinBounds = this.validateWithinBounds(
						this.coordinates.x,
						this.coordinates.y
					)
				}
			},

			validateCoo(coo) {
				if (!(coo != null && coo.length != 0 && coo >= 0 && coo % 1 == 0)) {
					this.active = true
					this.errorMessage =
						'Please enter valid initial coordinates or size of square. Only positive and decimal numbers accepted.'
					this.errorType = 'is-danger'
					return false
				} else {
					return true
				}
			},

			validateWithinBounds(x, y) {
				return !(y >= this.cooY || y < 0 || x < 0 || x >= this.cooX)
			},

			validateCommands(commArr) {
				if (
					commArr.findIndex(
						(comm) => comm !== 'L' && comm !== 'A' && comm !== 'R'
					) !== -1
				) {
					this.active = true
					this.errorMessage = 'Invalid command'
					this.errorType = 'is-danger'
					return false
				} else {
					return true
				}
			},
		},

		beforeMount() {
			this.errorMessage = this.defaultMsg // sets default message as note
		},
	}
</script>

<style scoped>
	form {
		max-width: 800px;
	}
	input::-webkit-outer-spin-button,
	input::-webkit-inner-spin-button {
		-webkit-appearance: none;
		margin: 0;
	}
	input[type='number'] {
		-moz-appearance: textfield;
	}
	.card {
		margin-top: 50px;
	}
	.card h2 {
		margin-bottom: 50px;
	}
	.red {
		color: rgba(194, 78, 78, 0.801);
	}
</style>
