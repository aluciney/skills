import type { FastifyError } from 'fastify'
import fp from 'fastify-plugin'
import { hasZodFastifySchemaValidationErrors, isResponseSerializationError } from 'fastify-type-provider-zod'
import { env } from '@/env'
import type { FastifyTypedInstance } from '@/types'

const isProd = env.NODE_ENV === 'production'

// Agrupa os erros de validação por `path`, acumulando as mensagens em uma array de strings
const groupIssues = (issues: Array<{ path: string; message?: string }>) => {
	const grouped = new Map<string, string[]>()

	for (const { path, message } of issues) {
		if (!message) continue
		const messages = grouped.get(path) ?? []
		messages.push(message)
		grouped.set(path, messages)
	}

	return Array.from(grouped, ([path, messages]) => ({ path, messages }))
}

export const errorHandlerPlugin = fp(async (app: FastifyTypedInstance) => {
	// Frase canônica do status HTTP ("Bad Request", "Internal Server Error", ...) via @fastify/sensible
	const reason = (statusCode: number) => app.httpErrors.getHttpError(statusCode as 400).message

	app.setErrorHandler((error, request, reply) => {
		// 1. Erro de validação do request (body, query, params, headers) via Zod
		if (hasZodFastifySchemaValidationErrors(error)) {
			return reply.status(400).send({
				statusCode: 400,
				error: reason(400),
				message: 'Erro de validação nos dados enviados',
				issues: groupIssues(
					error.validation.map((issue) => ({
						path: issue.instancePath.replace(/^\//, '').split('/').filter(Boolean)[0],
						message: issue.message,
					})),
				),
			})
		}

		// 2. Erro ao serializar a resposta contra o schema Zod (falha interna)
		if (isResponseSerializationError(error)) {
			request.log.error({ err: error }, 'A resposta não corresponde ao schema definido')
			return reply.status(500).send({
				statusCode: 500,
				error: reason(500),
				message: 'A resposta do servidor não corresponde ao formato esperado',
				...(isProd
					? {}
					: {
							issues: groupIssues(
								error.cause.issues.map((issue) => ({
									path: issue.path.map(String)[0],
									message: issue.message,
								})),
							),
						}),
			})
		}

		const fastifyError = error as FastifyError & { expose?: boolean }
		const statusCode = fastifyError.statusCode ?? 500
		const expose = fastifyError.expose ?? statusCode < 500

		// Erros não expostos (5xx) são logados por completo
		if (!expose) {
			request.log.error({ err: fastifyError }, 'Erro interno não tratado')
		}

		return reply.status(statusCode).send({
			statusCode,
			error: reason(statusCode),
			// mensagens client-safe são repassadas; as ocultas só aparecem fora de produção
			message: expose || !isProd ? fastifyError.message : reason(statusCode),
		})
	})
})
