# Created by: Chao Shin <quakelee@cn.FreeBSD.org>
# $FreeBSD$

PORTNAME=	pnetcdf
PORTVERSION=	1.6.1
PORTREVISION=	2
CATEGORIES=	science parallel
MASTER_SITES=	http://cucis.ece.northwestern.edu/projects/PnetCDF/Release/ \
		LOCAL/sunpoet
DISTNAME=	parallel-netcdf-${PORTVERSION}

MAINTAINER=	sunpoet@FreeBSD.org
COMMENT=	Library providing high-performance I/O

OPTIONS_DEFINE=	DOCS
OPTIONS_SINGLE=	MPI
OPTIONS_SINGLE_MPI=	OPENMPI MPICH2
OPTIONS_DEFAULT=OPENMPI

CONFLICTS=	hdf-4.*

CFLAGS+=	-fPIC -DPIC
GNU_CONFIGURE=	yes
MAKE_JOBS_UNSAFE=	yes
USE_PERL5=	build
USES=		fortran gmake perl5 tar:bzip2

NOPIE_PORTS=	yes
NORELRO_PORTS=	yes

PORTDOCS=	COPYRIGHT INSTALL README

MPICH2_CONFIGURE_ON=	--with-mpi=${LOCALBASE}
MPICH2_LIB_DEPENDS=	libmpich.so:net/mpich2
OPENMPI_CONFIGURE_ON=	--with-mpi=${LOCALBASE}/mpi/openmpi
OPENMPI_LIB_DEPENDS=	libmpi.so:net/openmpi

post-patch:
	@${FIND} ${WRKSRC} -name Makefile.in | ${XARGS} ${REINPLACE_CMD} -e '/$$(INSTALL.*$$(...DIR)/ s|$$(...DIR)|$$(DESTDIR)&|'

post-install:
	${MKDIR} ${STAGEDIR}${DOCSDIR}/
	cd ${WRKSRC}/ && ${INSTALL_DATA} ${PORTDOCS} ${STAGEDIR}${DOCSDIR}/
	${STRIP_CMD} ${STAGEDIR}${PREFIX}/bin/*

.include <bsd.port.mk>
