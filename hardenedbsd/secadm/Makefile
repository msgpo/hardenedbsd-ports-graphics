# $FreeBSD$

PORTNAME=	secadm
PORTVERSION=	0.3
PORTREVISION?=	2
CATEGORIES=	hardenedbsd

MAINTAINER=	shawn.webb@hardenedbsd.org
COMMENT=	HardenedBSD Security Administration

LIB_DEPENDS=	libucl.so:${PORTSDIR}/textproc/libucl

LIBDIR?=	${PREFIX}/lib

MAKE_JOBS_UNSAFE=	yes

USE_GITHUB=	yes
GH_ACCOUNT=	HardenedBSD
GH_PROJECT=	secadm
GH_TAGNAME=	162c5d1

.if defined(KMOD)
MAKE_ENV+=	WITHOUT_CLI=YES
.else
MAKE_ENV+=	WITHOUT_KMOD=YES
.endif

.include <bsd.port.pre.mk>

.if ${OPSYS} == FreeBSD && ${OSVERSION} < 1100000
LIB_DEPENDS+=	libxo.so:${PORTSDIR}/devel/libxo
.endif

.include <bsd.port.post.mk>

