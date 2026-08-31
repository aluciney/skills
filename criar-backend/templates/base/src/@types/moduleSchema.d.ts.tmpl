import type { z } from 'zod'

type HttpStatus = 200 | 201 | 400 | 401 | 404 | 500

declare global {
	interface ModuleSchema {
		[key: string]: {
			Body?: z.ZodTypeAny
			Query?: z.ZodTypeAny
			Params?: z.ZodTypeAny
			Response: {
				[key in HttpStatus]?: z.ZodTypeAny
			}
		}
	}
	type InferModuleSchema<T> = {
		Body: T extends { Body: z.ZodTypeAny } ? z.infer<T['Body']> : never

		Querystring: T extends { Query: z.ZodTypeAny } ? z.infer<T['Query']> : never

		Params: T extends { Params: z.ZodTypeAny } ? z.infer<T['Params']> : never

		Reply: T extends { Response: infer R }
			? {
					[K in keyof R]: R[K] extends z.ZodTypeAny ? z.infer<R[K]> : never
				}
			: never
	}
}
