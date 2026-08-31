import { Sema } from 'async-sema'

export class InstrumentedSema {
	private sema: Sema
	private capacity: number
	private inUse = 0
	private waiting = 0

	constructor(capacity: number) {
		this.capacity = capacity
		this.sema = new Sema(capacity)
	}

	async acquire() {
		this.waiting++

		await this.sema.acquire()

		this.waiting--
		this.inUse++

		let released = false

		return () => {
			if (released) return
			released = true

			this.inUse--
			this.sema.release()
		}
	}

	stats() {
		return {
			capacity: this.capacity,
			inUse: this.inUse,
			free: this.capacity - this.inUse,
			waiting: this.waiting,
		}
	}
}
