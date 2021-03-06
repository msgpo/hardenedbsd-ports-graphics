# Created by: Stas Timokhin <devel@stasyan.com>
# $FreeBSD$

PORTNAME=	openomf
PORTVERSION=	0.6.5
CATEGORIES=	games
MASTER_SITES=	http://www.omf2097.com/pub/files/omf/:data
DISTFILES+=	omf2097.rar:data
EXTRACT_ONLY=	${DISTFILE_shadow} ${DISTFILE_libdumb} \
		${DISTNAME}.tar.gz

MAINTAINER=	devel@stasyan.com
COMMENT=	Rewritten engine for One Must Fall 2097

LIB_DEPENDS=	libconfuse.so:devel/libconfuse \
		libpng.so:graphics/png \
		libargtable2.so:devel/argtable \
		libopenal.so:audio/openal \
		libenet.so:net/enet
EXTRACT_DEPENDS=	unrar:archivers/unrar

USE_GITHUB=	yes
GH_TUPLE=	omf2097:openomf:c36c421 \
		omf2097:libShadowDive:b2c34a6:shadow \
		kode54:dumb:238f23c:libdumb

USES=		cmake compiler:c++11-lang gettext
USE_SDL=	sdl2

NOPIE_PORTS=	yes

DESKTOP_ENTRIES="OpenOMF" "" \
		"${PREFIX}/share/openomf/openomf_icon.png " \
		"openomf" "Game;ArcadeGame;" false

post-extract:
	${RMDIR} ${WRKSRC}/external/libShadowDive
	${LN} -s ${WRKSRC_shadow} ${WRKSRC}/external/libShadowDive
	${LN} -s ${WRKSRC_libdumb}/dumb ${WRKSRC}/external/dumb
	${MKDIR} ${WRKDIR}/omf
	cd ${WRKDIR}/omf && unrar e -idq ${DISTDIR}/omf2097.rar

post-patch:
	${REINPLACE_CMD} -e 's,%%PREFIX%%,"${PREFIX}",g' \
		${WRKSRC}/src/resources/pathmanager.c

do-install:
	${INSTALL_PROGRAM} ${WRKSRC}/openomf ${STAGEDIR}${PREFIX}/bin
	@${MKDIR} ${STAGEDIR}${DATADIR}
	${INSTALL_DATA} ${WRKSRC}/resources/openomf_icon.png ${STAGEDIR}${DATADIR}
	${INSTALL_DATA} ${WRKSRC}/resources/openomf.bk ${STAGEDIR}${DATADIR}
	(cd ${WRKDIR}/omf && ${COPYTREE_SHARE} . ${STAGEDIR}${DATADIR})

.include <bsd.port.mk>
